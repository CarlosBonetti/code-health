# code-health

This repository is one agent skill, `skills/code-health/SKILL.md`, packaged as a Claude Code plugin.

## Principles

- Say the goal and why it matters, not how to reach it. Agents keep getting more capable and creative, and a clear goal
  lets them find a better route than steps written for a weaker one; prescribed steps age into a ceiling.
- Keep modes and rules generic, each with a focused goal. When a run goes wrong, sharpen the goal or the judgement it
  calls for before adding a step, a checklist or a special case.

## Conventions

- `SKILL.md` is a prompt: every line costs every run that loads it. Prefer tightening a rule to adding one, and keep
  each mode's section to its goal, its judgement and its "Report also" line.
- The frontmatter `description` decides when the skill triggers. Keep the mode list in it in step with the `Modes`
  section and the README's table.
- A user-visible change bumps `version` in `.claude-plugin/plugin.json` and gets a `CHANGELOG.md` entry, in the same
  commit.
- Check `.claude-plugin/*.json` still parses (`jq . .claude-plugin/*.json`) and, after a change to a mode, try it on a
  real repository before committing.
