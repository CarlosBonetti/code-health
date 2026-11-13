# code-health

An agent skill that leaves a codebase healthier in **one focused change a reviewer can read in one sitting**. Each run
goes deep on one mode and one area, proves what it changes, gets the change reviewed, and writes a report the next run
picks up from.

```
/code-health <mode> [area] [auto]
```

## Modes

| Mode           | Goal                                                                              |
| -------------- | --------------------------------------------------------------------------------- |
| `tests`        | A suite that catches real breakage and is cheap to maintain                       |
| `abstractions` | One implementation per idea, and abstractions that earn their keep                |
| `dead-code`    | Delete what nothing uses, with evidence for each deletion                         |
| `dependencies` | No known advisories, and no dependency left far behind                            |
| `security`     | No vulnerability an attacker could realistically use, each proven by a test       |
| `performance`  | Fast where users and costs feel it, measured before and after                     |
| `setup`        | Setup and deploy instructions that work as written                                |
| `docs`         | Docstrings, comments and docs that tell the truth about the code                  |
| `structure`    | A folder layout that is easy to navigate, with boundaries that hold               |
| `onboarding`   | A fresh agent gets oriented fast, judged from how recent agent sessions fared     |

- **area** narrows the run to a module, folder or span of sessions. Left out, the skill picks one recent runs haven't
  covered and says why.
- **auto** runs unattended: no questions, the conservative option where a call is ambiguous.

## Examples

```
/code-health tests src/billing
/code-health dead-code
/code-health dependencies auto
/code-health onboarding
```

## What a run does

1. Reads the project's agent instructions and runs its checks for a baseline.
2. Finds and ranks candidates for the mode, and (interactively) waits for you to choose.
3. Makes one focused change, keeping behaviour unless the mode exists to change it.
4. Gets the change reviewed by a worker without its context, and exercised end to end when it can reach runtime.
5. Writes a report to `.context/code-health/<mode>/<YYYY-MM-DD>.md`, and opens a PR only when asked.

## Principles

Each mode states a goal and why it matters, not the steps to reach it. Agents keep getting more capable and creative,
so a focused goal gets better work out of them than an ever more refined procedure, and it doesn't age as models
improve. Modes stay generic for the same reason: a fix to a mode sharpens its goal or judgement rather than adding
steps or special cases.

## Requirements

The skill is written for workers in **the tower**, a local HQ for Claude Code sessions: it uses the `tower` command to
find itself, hire reviewers, leave notes and show its report. Run as a tower **policy**, it becomes standing work on a
schedule, each run reading what the last one left.

Outside the tower the modes and rules still apply, but the review, end-to-end and reporting steps that call `tower`
won't run as written.

## Install

As a Claude Code plugin, from this repository's marketplace:

```
/plugin marketplace add CarlosBonetti/code-health
/plugin install code-health@code-health
```

Or as a personal skill, by linking it into `~/.claude/skills`:

```sh
git clone https://github.com/CarlosBonetti/code-health.git
ln -s "$PWD/code-health/skills/code-health" ~/.claude/skills/code-health
```

## Repository layout

```
.claude-plugin/
  plugin.json        plugin manifest (name, version, author)
  marketplace.json   lets /plugin marketplace add install it
skills/code-health/
  SKILL.md           the skill itself: frontmatter, rules and the ten modes
CHANGELOG.md
LICENSE
```

## Contributing

Issues and pull requests are welcome. A change to a mode should say what went wrong on a real run without it, and
follow the [principles](#principles). See [`CLAUDE.md`](CLAUDE.md) for the conventions an agent (or you) should follow
when editing the skill, and record every user-visible change in [`CHANGELOG.md`](CHANGELOG.md).

## License

[MIT](LICENSE)
