---
name: code-health
description: Leave a codebase healthier in one focused, reviewable change. Modes - `tests` (a suite that catches real breakage and is cheap to maintain), `abstractions` (one implementation per idea, abstractions that earn their keep), `dead-code` (delete what nothing uses), `dependencies` (no known advisories, no dependency left far behind), `security` (no vulnerability an attacker could realistically use), `performance` (fast where users and costs feel it), `setup` (setup and deploy instructions that work as written), `docs` (docstrings, comments and docs that tell the truth about the code), `structure` (a folder layout that's easy to navigate, with boundaries that hold) and `onboarding` (a fresh agent gets oriented fast, judged from how recent agent sessions fared). Use when started as `/code-health <mode> [area] [auto]`, or when asked to review or clean up tests, find duplication or over-abstraction, remove dead or unused code, do a dependency audit or upgrade, find and fix security vulnerabilities, find and fix slow or costly code paths, check that setup or deploy docs still work, fix stale comments or docs, review a repository's folder structure or layering, or find what slows agents down in a codebase from their session logs. Written for tower workers, by hand or as the prompt of a policy run.
---

# Code health

Leave the codebase better than you found it, in **one focused change a reviewer can read in one sitting**. Go deep on one area per run rather than sweeping everything.

You are a worker in the tower. Run `tower whoami` first, and load `tower:handbook` before you hire anyone, write notes or end yourself.

## Arguments

`/code-health <mode> [area] [auto]`

- **mode:** `tests`, `abstractions`, `dead-code`, `dependencies`, `security`, `performance`, `setup`, `docs`, `structure` or `onboarding`. Ask if it's missing and not obvious.
- **area:** where to focus. Without one, pick an area recent runs haven't covered, where the mode is likely to pay off, and say why.
- **auto:** run unattended. A policy run (`tower whoami` says so) is always unattended.

Interactively, present the ranked findings and the change you propose, and wait for the user to choose before you edit. Unattended, never stop to ask: where a call is genuinely ambiguous, take the conservative option and say so in the report.

Whoever invoked you (the user, or the policy whose prompt loaded this skill) may add rules: a report path, PR steps, project exclusions, how to run the app. Theirs win.

## Before you start

- Read the project's agent instructions and follow them all run. Use its own toolchain; the checks CI runs are the bar.
- As a policy run, read `tower runs` for what earlier runs covered and left open, and don't redo their work.
- Install dependencies and run the suites once for a baseline. Report anything already failing; don't fix it unless the mode makes it yours.

## Rules for every mode

- One focused change; everything else goes on a checklist for later.
- Behaviour doesn't change, except where the mode exists to change it. Existing tests keep asserting what they asserted.
- A bug outside the mode's job gets reported with file and line, not fixed in passing.
- Don't change public contracts or database schemas, run migrations, edit generated files, or touch what the project puts off limits. List those findings.
- Tests never call real models, paid APIs or the network.
- Every change carries its evidence in the report.
- Finish green on every check CI would run for what you touched. If something stays red, say which check and why.
- Finding nothing worth changing is a valid result. Don't churn to have something to show.
- Commit and open PRs only when asked, on your own branch, never the default one.

## Modes

### `tests`

Goal: a suite that catches real breakage, costs little to maintain, and where each test sits where someone changing the code it covers would look for it. Add tests where untested behaviour is easy to break and costly when broken. Remove tests that no longer earn their place. Simplify the rest, fix what makes them flaky, slow or brittle, and move tests that drifted from the code they cover. Let coverage guide you, not decide for you.

Never change production code to make a test pass. A test that exposes a real bug is marked expected-to-fail with a reason, and the bug is reported.

Report also: tests added, removed and simplified; coverage for the area before and after.

### `abstractions`

Goal: one implementation per idea, and abstractions that earn their keep. Find logic written several times in different shapes, abstractions that only add indirection, and boundaries callers have to reach around. Rank by how much code and future risk each fix removes, and fix the top two.

Unify only what you can show is equivalent, or whose differences you can name and keep. If one version looks like a bug next to the others, report it rather than fixing it inside the merge. Add tests first where the code being unified has none.

Report also: the duplicated versions side by side, the one you kept, and lines removed.

### `dead-code`

Goal: delete code nothing uses. Code is dead only when you can show nothing reaches it, dynamically included: registries, string lookups, framework conventions, entry points, generated clients, outside consumers. Delete only high-confidence items; list the rest with why you weren't sure. Code used only by its own tests goes with those tests.

Report also: each deletion with its evidence, and lines removed.

### `dependencies`

Goal: no known advisories, and dependencies that never fall so far behind that catching up becomes a project of its own. Each run does one of two jobs, in this order: fix the advisories that can be fixed, or, when none can, make one planned upgrade.

**Advisories.** Run the ecosystem's audit for every lockfile. Prefer updating the direct dependency within its major, then refreshing the transitive one in place. Use an override only when nothing else works, scoped so it never lifts a package across a major version, and drop overrides whose parent has since shipped a fix. Before calling an advisory blocked upstream, check whether it can be reached. Earlier runs' reports list the advisories blocked upstream: check one again only when the package blocking it has published a release since.

**Planned upgrades.** A major upgrade is one package per run. In a 0.x package, a minor version counts as a major. Pick one from the outdated list earlier runs left, preferring those your caller approved. Unattended, take a major only when your caller approved it or it is the only fix for a high or critical advisory. Otherwise, list it. Read its changelog for breaking changes and check each one against how the code uses the package.

Lockfiles change only through the package manager. Raise a manifest's lower bound when the lock moves past an affected version, so a later relock can't go back to it. Leave model names alone; list them if they look stale. Run end-to-end checks on the paths the upgraded packages reach.

Report also: each advisory with its ID, severity, whether it can be reached, and versions before and after; the blocked list, with what each item waits on upstream; for a major, the breaking changes and how each applies to this code; the remaining outdated majors, ranked by risk.

### `security`

Goal: no vulnerability an attacker could realistically use. Find the few that matter, prove them and fix the worst. Don't produce a list of everything that could be hardened. Known advisories in dependencies belong to `dependencies`.

Judge each candidate on reachability (who outside the trust boundary can trigger it), exploitability (whether the attack works against the code as written, past the defences already in place) and blast radius (what the attacker gains). Keep only findings that hold up on all three, and rank them by the combination. A candidate is a hypothesis until you've traced it from attacker-controlled input to the dangerous operation. Try to refute your top findings before you fix them. Ignore theoretical, pedantic and low-impact issues. Count them, don't list them.

Prove each finding with a test that runs the attack locally and fails, then fix the top one or two at the root. Attack only your own checkout, never a live environment or a third party. A fix that changes an API contract, the auth model or how a running deploy behaves is a proposal: list it. Keep exploit details out of anything public until the fix ships.

Report also: each finding with its trace (file and line), its three judgements and the proving test; the refuted candidates and why they failed; findings left for later, ranked.

### `performance`

Goal: the app is fast where users and costs feel it. Find the few slowdowns that matter, prove them with measurements and fix the worst. Don't micro-optimize code that runs rarely or is already cheap.

Judge each candidate on how often it runs, what it costs each time (latency, compute, memory, queries, paid calls) and how many users or runs it touches. Keep only findings where that combination is large, and rank by it. A candidate is a guess until you've measured it on realistic input: profile, time or count before you change anything. Ignore wins too small to notice. Count them, don't list them.

Fix the top one or two at the cause, without changing behaviour or results. Add a test where the fix could quietly regress (a query count, a call count, a bound on work). Measure again with the same setup and keep only fixes that clearly pay off. A fix that needs a schema change, new infrastructure or a different trade-off for users (caching stale data, lower quality, a cheaper model) is a proposal: list it.

Report also: each finding with where the time or cost goes (file and line); the measurements before and after, and how you took them; the candidates that measured fine; findings left for later, ranked.

### `setup`

Goal: anyone who follows the project's setup and deploy instructions as written ends up with a working app. Pick one path per run (local development, containers, or one deploy target) and follow it step by step from a clean start. Use throwaway databases, clusters and spare ports, and remove them afterwards.

Where the instructions and the config or code disagree, work out which one is right:

- When the code is right, fix the instructions.
- When the instructions describe what was intended, fix the config, as long as the fix is small and changes nothing for anyone already running the app.
- Anything that would change how a running deploy behaves is a proposal: list it, don't make it.

Read live environments only, and only when your caller allows it. Never change one. Never write a secret into a file, an image or a report. A step you can't run (it needs an account, a platform or an approval you don't have) may still be corrected from that platform's own documentation, but mark it unverified.

Report also: the path you followed, and each step's result before and after; the steps you couldn't run, and why; the proposals that would change behaviour.

### `docs`

Goal: docs that tell the truth about the code. That covers docstrings, comments, and the prose that describes what the code is and does: architecture and integration docs, module READMEs, agent instruction files. Instructions for installing, running or deploying belong to `setup`. Work through one module or one set of docs per run.

The bar is accuracy, not taste:

- **Fix** what is false or misleading: a comment that contradicts the code, a reference to something removed or renamed, a parameter, return value or error described wrongly, behaviour that has since changed.
- **Delete** comments that only restate the code, and pointers to things that are gone.
- **Add** only where a reader would otherwise get it wrong: a non-obvious invariant, the reason code looks odd. Never add docs for coverage.

Match the surrounding style and comment density. In agent instruction files, correct facts such as commands, paths and names. The rules in those files are the project owner's: list the ones that look outdated, and don't change them. Leave prompts, agent skills and generated files alone, and list what's wrong in them.

Report also: each change with the code that shows the old text was wrong (file and line); counts fixed, deleted and added.

### `structure`

Goal: a layout where someone new can guess where code lives, files that change together sit together, and the layers' boundaries hold. Judge both whether the code follows the repo's conventions and whether those conventions are the right ones for this codebase and stack. Questioning the layout is part of the job. Where individual tests sit is the `tests` mode's job; whether the test convention itself is right is yours. Tests of the code you move follow it, to where the project's test convention puts them; the `tests` mode's job is tests that drifted on their own.

An improvement needs evidence that it solves a real problem, not just that it looks cleaner. When the better structure differs from the current one, write it down as a target layout with its rationale and migrate one piece toward it. Never leave a third convention next to two existing ones.

Move one cluster per run. A path is referenced in more places than imports (routing, config, CI, docs, outside consumers), so a move is done only when every reference follows it. Leave alone files that an open PR or an on-duty worker (`tower agents`) is changing.

Report also: the conventions you found, the target layout if you proposed one, the tree before and after, and remaining findings, with adherence and improvement findings kept apart.

### `onboarding`

Goal: an agent new to the codebase gets oriented fast. It finds the right file on the first try, doesn't trip on the same tooling twice, and doesn't work out again what an earlier session already worked out. Every session pays that cost before any real work, and the code alone can't show where it goes, so the evidence is how recent agent sessions in this project actually fared, read from their logs. The area is a span of sessions, by default those since the last `onboarding` run.

A finding is a pattern, not an anecdote: something several sessions stumbled on, or one lost many steps to. Typical causes are code that is hard to guess at, knowledge that lives only in sessions that have ended, and instructions that are missing, wrong, contradictory or never reach the agent. Rank by the steps they cost across sessions.

Fix the top one or two at the cause, in the repo. The agent instruction file loads into every session, so each line costs every future one: prefer a short doc it links to, and cut what no longer earns its place. Adding facts to it is yours; changing a rule in it is a proposal. What another mode owns goes on the list for that mode.

Causes outside the repo (the user's private instructions, memory or settings, the runtime's configuration, a policy's prompt) are proposals: write the exact change for the user. Session logs carry briefs, client details and sometimes secrets, so quotes from them stay in the report, never in a commit or PR.

A fix shows in later sessions, not in the diff. Measure each finding's cost in a way the next run can repeat, so each run says whether the last one's fixes worked.

Report also: the sessions read and their date range; each finding with the sessions that show it and what it cost; the measurements beside the previous run's; the changes proposed outside the repo.

## Review and end-to-end checks

Once the checks pass, get the change judged by someone without your context.

**Review.** Leave a `tower note` on your thread with what the reviewer needs: the mode, your evidence, and what you'd look hardest at. Then run `tower review <your callsign> tell`. End your turn to wait; the notes arrive as your next prompt. Act on each one and answer it with `tower note re n<k>`, saying what changed or why it stays. Hire a second review if the fixes were substantial.

**End-to-end, when the change can reach runtime** and the project has a way to exercise it. Hire a worker with `tower hire` to run the affected paths against your checkout. It starts with none of your context or uncommitted work, so the brief carries everything:

- who you are, and to report back by `SendMessage` to your **message as** name
- your checkout's path, which it runs from without editing
- how to run the app without disturbing anyone else's instance
- the paths to exercise, and what "working" means for each

Hold your edits until it reports, fix what failed, and send it home with `tower home`. If the hire is refused, run the paths yourself and say the check wasn't independent. Skip end-to-end for changes that can't reach runtime, and say why.

## Report

Write a short markdown report, at the path your caller names or else `.context/code-health/<mode>/<YYYY-MM-DD>.md` (outside the repo if it doesn't ignore `.context/`). `tower show` it, again after every change, and leave a `tower note` pointing at it and at the choices worth a second look.

It covers: the area and why; the baseline; each change with its evidence, plus the mode's extras; bugs found; what's left for later, with confidence; final checks; the review and end-to-end outcomes, with the workers' callsigns; and what the next run should look at. When nothing was worth changing, say what you checked and why it was fine, and skip the review.

## Pull request

When your caller asks for one and the run produced something meaningful: commit on your branch in the project's commit convention, push with `git push -u origin <branch>`, and open the PR with `gh pr create` against the base and template the project names. Include the bugs found and the checklist. Never merge.

## Final answer and ending the run

End with: the PR link or "no PR" and why, the area, what changed, the report path, bugs found, the review and end-to-end outcomes, and the next area. A policy's next run reads this, so write it for that reader.

As a policy run, with no PR: check that your hires have gone home, then run `tower home <your callsign>` in the same turn as your final answer. With a PR: stay on duty. The user comes back to this worker to adjust it; make their changes in the same worktree, push to the same branch, and answer new notes on your thread.
