---
name: code-health
description: Leave a codebase healthier in one focused, reviewable change. Modes - `tests` (a suite that catches real breakage and is cheap to maintain), `abstractions` (one implementation per idea, abstractions that earn their keep), `dead-code` (delete what nothing uses), `dependencies` (no known advisories, no dependency left far behind), `security` (no vulnerability an attacker could realistically use), `performance` (fast where users and costs feel it), `setup` (setup and deploy instructions that work as written), `docs` (docstrings, comments and docs that tell the truth about the code), `structure` (a folder layout that's easy to navigate, with boundaries that hold) and `onboarding` (a fresh agent gets oriented fast, judged from how recent agent sessions fared). Use when started as `/code-health <mode> [area] [auto]`, or when asked to review or clean up tests, find duplication or over-abstraction, remove dead or unused code, do a dependency audit or upgrade, find and fix security vulnerabilities, find and fix slow or costly code paths, check that setup or deploy docs still work, fix stale comments or docs, review a repository's folder structure or layering, or find what slows agents down in a codebase from their session logs. Runs by hand, or unattended on a schedule.
---

# Code health

Leave the codebase better than you found it, in **one focused change a reviewer can read in one sitting**. A small change gets a real review, and a run that goes deep on one area finds what a sweep misses.

Some steps need help from outside your context: a reviewer, a tester, a way to show the user your report. Use what your environment offers (subagents, separate sessions, a review tool). If you're running in the tower (a `tower` command is on your PATH), you are a tower worker: run `tower whoami` first, and load `tower:handbook` before you hire anyone, write notes, show a file or end yourself; it has the current commands for each, and its scheduled runs are called policies.

## Arguments

`/code-health <mode> [area] [auto]`

- **mode:** `tests`, `abstractions`, `dead-code`, `dependencies`, `security`, `performance`, `setup`, `docs`, `structure` or `onboarding`. Ask if it's missing and not obvious.
- **area:** where to focus. Without one, pick an area recent runs haven't covered, where the mode is likely to pay off, and say why.
- **auto:** run unattended. A scheduled or headless run is always unattended.

Interactively, present the ranked findings and the change you propose, and wait for the user to choose before you edit. Unattended, never stop to ask: where a call is genuinely ambiguous, take the conservative option and say so in the report.

Whoever invoked you (the user, or the scheduled job whose prompt loaded this skill) may add rules: a report path, PR steps, project exclusions, how to run the app. Theirs win.

## Before you start

- Read the project's agent instructions and follow them all run. Use its own toolchain; the checks CI runs are the bar.
- Don't redo earlier runs' work. Their reports and your caller's brief say what they covered and left open.
- Before you change anything, know which checks already pass, so you can show your change broke nothing. Report what was already failing; don't fix it unless the mode makes it yours.

## Rules for every mode

- One focused change; everything else goes on a list for later.
- Behaviour doesn't change, except where the mode exists to change it, so a green suite tells the reviewer nothing broke. Existing tests keep asserting what they asserted.
- A bug outside the mode's job gets reported with file and line, not fixed in passing, so the change stays what it claims to be.
- Some changes are the owner's decision, not yours: a public contract, a database schema, a migration, the auth model, how a running deploy behaves, a trade-off users would feel, generated files, and what the project puts off limits. Make them proposals: list each with the change you'd make.
- Tests never call real models, paid APIs or the network: that costs money, flakes, and fails for reasons unrelated to the code.
- Every change carries its evidence in the report. The reviewer judges the evidence, not your confidence.
- Finish green on every check CI would run for what you touched. If something stays red, say which check and why.
- Finding nothing worth changing is a valid result. Don't churn to have something to show.
- Commit and open PRs only when asked, on your own branch, never the default one.

## Modes

### `tests`

Goal: a suite that catches real breakage, costs little to maintain, and where each test sits where someone changing the code it covers would look for it. Add tests where untested behaviour is easy to break and costly when broken. Remove tests that no longer earn their place. Simplify the rest, fix what makes them flaky, slow or brittle, and move tests that drifted from the code they cover. Let coverage guide you, not decide for you.

Tests describe the code as it is, so production code doesn't change to make one pass. A test that exposes a real bug is marked expected-to-fail with a reason, and the bug is reported.

Report also: tests added, removed and simplified, and how you know the new ones catch breakage; coverage for the area before and after, when the project already measures coverage.

### `abstractions`

Goal: one implementation per idea, and abstractions that earn their keep. Find logic written several times in different shapes, abstractions that only add indirection, and boundaries callers have to reach around. Fix what removes the most code and future risk, where the result is simpler for whoever maintains it, not only shorter.

A merge must not change behaviour, so unify only what you can show is equivalent, or whose differences you can name and keep. If one version looks like a bug next to the others, report it rather than fixing it inside the merge.

Report also: the duplicated versions side by side, the one you kept, how you showed they're equivalent, and lines removed.

### `dead-code`

Goal: delete code nothing uses. Code is dead only when you can show nothing reaches it, dynamically included: registries, string lookups, framework conventions, entry points, generated clients, outside consumers. A wrong deletion breaks something no test may cover, so delete only high-confidence items and list the rest with why you weren't sure. Code used only by its own tests goes with those tests.

Report also: each deletion with its evidence, and lines removed.

### `dependencies`

Goal: no known advisories, and dependencies kept close enough to current that catching up never becomes a project of its own, without breaking anything on the way. Fix the advisories that can be fixed first; when none are left, catch up on what's outdated.

Updates within a major version are yours to make. A major (in a 0.x package, a minor) can break code, so take one only when your caller approved it or it is the only fix for a high or critical advisory: one package per run, with each breaking change in its changelog checked against how this code uses the package. List the other majors.

Keep each fix small and durable: lockfiles change only through the package manager, an override is a last resort that never lifts a package across a major and goes once nothing needs it, and a later relock must not be able to bring a fixed advisory back. An advisory is blocked only when it can be reached and nothing within reach fixes it; say what it waits on upstream. Run end-to-end checks on the paths the updated packages reach.

Report also: each advisory with its ID, severity, whether it can be reached, and versions before and after; the blocked list, with what each item waits on upstream; for a major, the breaking changes and how each applies to this code; the remaining outdated majors, ranked by risk.

### `security`

Goal: no vulnerability an attacker could realistically use. Find the few that matter, prove them and fix the worst. Don't produce a list of everything that could be hardened. Known advisories in dependencies belong to `dependencies`.

Judge each candidate on reachability (who outside the trust boundary can trigger it), exploitability (whether the attack works against the code as written, past the defences already in place) and blast radius (what the attacker gains). Keep only findings that hold up on all three, and rank them by the combination. A candidate is a hypothesis until you've traced it from attacker-controlled input to the dangerous operation. Try to refute your top findings before you fix them. Ignore theoretical, pedantic and low-impact issues. Count them, don't list them.

Prove each finding you fix with a test that runs the attack locally and fails without the fix, so the fix is shown to work and stays fixed. Fix at the root. Attack only your own checkout, never a live environment or a third party. Keep exploit details out of anything public until the fix ships.

Report also: each finding with its trace (file and line), its three judgements and the proving test; the refuted candidates and why they failed; findings left for later, ranked.

### `performance`

Goal: the app is fast where users and costs feel it. Find the few slowdowns that matter, prove them with measurements and fix the worst. Don't micro-optimize code that runs rarely or is already cheap.

Judge each candidate on how often it runs, what it costs each time (latency, compute, memory, queries, paid calls) and how many users or runs it touches. Keep only findings where that combination is large, and rank by it. A candidate is a guess until you've measured it on realistic input. Ignore wins too small to notice. Count them, don't list them.

Fix at the cause, without changing behaviour or results, and measure again the same way; keep only fixes that clearly pay off. Where a fix could quietly regress, add a test that would catch it (a query count, a call count, a bound on work). Caching stale data, lowering quality or switching to a cheaper model are trade-offs users would feel.

Report also: each finding with where the time or cost goes (file and line); the measurements before and after, and how you took them; the candidates that measured fine; findings left for later, ranked.

### `setup`

Goal: anyone who follows the project's setup and deploy instructions as written ends up with a working app. Pick one path per run (local development, containers, or one deploy target) and follow it from a clean start, without disturbing anything already running or leaving anything behind.

Where the instructions and the config or code disagree, work out which one is right. When the code is right, fix the instructions. When the instructions describe what was intended, fix the config, as long as the fix is small and changes nothing for anyone already running the app.

Read live environments only, and only when your caller allows it. Never change one. Never write a secret into a file, an image or a report. A step you can't run (it needs an account, a platform or an approval you don't have) may still be corrected from that platform's own documentation, but mark it unverified.

Report also: the path you followed, and each step's result before and after; the steps you couldn't run, and why; the proposals that would change behaviour.

### `docs`

Goal: docs that tell the truth about the code. That covers docstrings, comments, and the prose that describes what the code is and does: architecture and integration docs, module READMEs, agent instruction files. Instructions for installing, running or deploying belong to `setup`. Work through one module or one set of docs per run.

The bar is accuracy, not taste. Fix or delete what is false, misleading, points at something gone, or only restates the code. Add only where a reader would otherwise get it wrong, such as a non-obvious invariant or the reason code looks odd: docs written for coverage are noise a reader has to get past. Match the surrounding style and comment density.

In agent instruction files, correct facts such as commands, paths and names. The rules in those files are the project owner's: list the ones that look outdated, and don't change them. Leave prompts, agent skills and generated files alone, and list what's wrong in them.

Report also: each change with the code that shows the old text was wrong (file and line); counts fixed, deleted and added.

### `structure`

Goal: a layout where someone new can guess where code lives, files that change together sit together, and the layers' boundaries hold. Judge both whether the code follows the repo's conventions and whether those conventions are the right ones for this codebase and stack. Questioning the layout is part of the job, including the test convention; where individual tests sit is the `tests` mode's.

An improvement needs evidence that it solves a real problem, not just that it looks cleaner. When the better structure differs from the current one, write it down as a target layout with its rationale and migrate one piece toward it. A third convention next to two existing ones makes the layout harder to guess, so never leave one.

Move one cluster per run, with its tests. A path is referenced in more places than imports (routing, config, CI, docs, outside consumers), so a move is done only when every reference follows it. Leave alone files that an open PR or another agent's in-flight work is changing.

Report also: the conventions you found, the target layout if you proposed one, the tree before and after, and remaining findings, with adherence and improvement findings kept apart.

### `onboarding`

Goal: an agent new to the codebase gets oriented fast. It finds the right file on the first try, doesn't trip on the same tooling twice, and doesn't work out again what an earlier session already worked out. Every session pays that cost before any real work, and the code alone can't show where it goes, so the evidence is how recent agent sessions in this project actually fared, read from their logs. The area is a span of sessions, by default those since the last `onboarding` run.

A finding is a pattern, not an anecdote: something several sessions stumbled on, or one lost many steps to. Typical causes are code that is hard to guess at, knowledge that lives only in sessions that have ended, and instructions that are missing, wrong, contradictory or never reach the agent. Rank by the steps they cost across sessions, and fix the costliest at the cause. What another mode owns goes on the list for that mode.

The repo is shared, so a fix there must help anyone working in this project. The agent instruction file loads into every session, so each line costs every future one: prefer a short doc it links to, and cut what no longer earns its place. Adding facts to it is yours; changing a rule in it is a proposal. What is particular to one user (their machine, shell, local services or habits) and causes outside the repo (the user's private instructions, memory or settings, the runtime's configuration, a scheduled job's prompt) are proposals: write the exact change for the user. Session logs carry briefs, client details and sometimes secrets, so quotes from them stay in the report, never in a commit or PR.

A fix shows in later sessions, not in the diff. Measure each finding's cost in a way the next run can repeat, so each run says whether the last one's fixes worked.

Report also: the sessions read and their date range; each finding with the sessions that show it and what it cost; the measurements beside the previous run's; the changes proposed outside the repo.

## Review and end-to-end checks

Once the checks pass, get the change judged by someone without your context: you can't see your own blind spots. Match the effort to what a mistake would cost. A change that can break something gets both checks below; one that can't (a few lines nothing reaches, a comment) may need neither, and the report says why.

**Review.** Get a reviewer that starts without your context, and give it what it needs: the mode, your evidence, and what you'd look hardest at. Answer every finding, saying what changed or why it stays, where the next reader will see it. If the fixes were substantial, get them reviewed again.

**End-to-end, when the change can reach runtime** and the project has a way to exercise it. Have a fresh agent run the affected paths against your checkout, so the check doesn't share your assumptions. It starts with none of your context or uncommitted work, so its brief carries everything: who you are and how to report back, your checkout's path (it runs from there without editing), how to run the app without disturbing anyone else's instance, and what "working" means for each path. Hold your edits until it reports, fix what failed, and release it. If you can't get one, run the paths yourself and say the check wasn't independent.

## Report

Write a short markdown report at the path your caller names, or else `.context/code-health/<mode>/<YYYY-MM-DD>.md` (outside the repo if it doesn't ignore `.context/`), and show it to the user. Its readers are the reviewer, who checks your evidence, and the next run, which picks up where you stopped. Give them what they need to verify each change and to continue: the area and why, each change with its evidence and the mode's extras, bugs found, what's left and how sure you are, and how the checks and reviews came out. When nothing was worth changing, say what you checked and why it was fine.

## Pull request

When your caller asks for one and the run produced something meaningful, open it the way the project does: its branch, commit convention, base and template. Include the bugs found and the list for later. Never merge; that's the owner's call.

## Final answer and ending the run

End with what the next reader needs: the PR link or "no PR" and why, the area, what changed, the report path, bugs found, the review and end-to-end outcomes, and the next area. A scheduled job's next run reads this.

A scheduled run with no PR ends once the agents it started have finished. With a PR, stay available: the user may come back to this session to adjust it, in the same worktree and branch. Wait to be prompted rather than watching for activity.
