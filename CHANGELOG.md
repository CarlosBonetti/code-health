# Changelog

All notable changes to this skill. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and
versions follow [Semantic Versioning](https://semver.org/): a new mode or rule is a minor, a renamed or removed mode or
a changed argument is a major.

## [0.3.0] - 2026-11-19

### Changed

- The skill works outside the tower: reviews and end-to-end checks use whatever fresh agent the environment offers,
  and the report is shown to the user however the environment allows. Tower-specific commands sit in one conditional
  paragraph that sends tower workers to `tower:handbook`.
- "Policy run" is now "scheduled run", and the end of a run no longer assumes tower workers' lifecycle: a
  scheduled run with no PR ends once its helpers have finished.

## [0.2.0] - 2026-11-13

### Changed

- Rules and modes state their goal and why, not the steps: shorter, with fixed counts ("fix the top two") and
  prescribed order dropped.
- One rule for what is the owner's decision (contracts, schemas, running deploys, trade-offs users feel), in place of
  the "is a proposal" line repeated across modes.
- The review, report, PR and ending sections say what each is for and leave the commands to `tower:handbook`, since
  the ones written here had gone stale. Review and end-to-end checks scale to what a mistake would cost.
- `dependencies` catches up on outdated packages within their major when no advisory is left, instead of stopping
  when no major is approved.
- `tests` reports coverage before and after only when the project already measures coverage.
- `onboarding` treats what is particular to one user's setup as a proposal, not a repo change.

### Removed

- Reading `tower runs` before starting, and the install-and-run baseline as a fixed step.

## [0.1.1] - 2026-11-13

### Changed

- The skill and marketplace descriptions no longer state how many modes there are, so adding or removing a mode
  doesn't leave them wrong.

## [0.1.0] - 2026-11-13

### Added

- First public release, with ten modes: `tests`, `abstractions`, `dead-code`, `dependencies`, `security`,
  `performance`, `setup`, `docs`, `structure` and `onboarding`.
- Claude Code plugin manifest and marketplace, so the skill installs with `/plugin`.
