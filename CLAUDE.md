# code-health

This repository is one agent skill, `skills/code-health/SKILL.md`, packaged as a Claude Code plugin.

- `SKILL.md` is a prompt: every line costs every run that loads it. Prefer tightening a rule to adding one, and keep
  each mode's section to its goal, its judgement and its "Report also" line.
- The frontmatter `description` decides when the skill triggers. Keep the mode list in it in step with the `Modes`
  section and the README's table.
- A user-visible change bumps `version` in `.claude-plugin/plugin.json` and gets a `CHANGELOG.md` entry, in the same
  commit.
- Check `.claude-plugin/*.json` still parses (`jq . .claude-plugin/*.json`) and, after a change to a mode, try it on a
  real repository before committing.
