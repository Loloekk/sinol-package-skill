# sinol-package-skill

A Claude Code plugin with one skill, `skills/sinol-package/`, for SIO2 task
packages. Part of three repositories (all kubat1234 on GitHub, usually checked
out side by side):

- **contest-template** — contest repository template: `contest.py`, `_templates/`;
- **sinol-gen** — `gen.h`, the ingen library;
- **sinol-package-skill** (this one) — the skill, with copies of both.

## Rules

- **Every push to `main` is a release.** `plugin.json` has no `version` on
  purpose (users get the commit SHA); never add one, never tag.
- **Generated, never edit by hand:** `skills/sinol-package/assets/`,
  `skills/sinol-package/references/sinol-gen/`, `skills/sinol-package/scripts/contest.py`.
  They are copies made by `bin/sync-assets.sh`; the `sync assets` workflow
  commits them to `main` daily (or `gh workflow run sync-assets.yml`). Change
  them in contest-template or sinol-gen instead.
- When contest-template or sinol-gen change behaviour, update the skill text
  that describes it (mostly `references/generator.md`, `contest-repo.md`).
- sinol-gen releases come from `SINOL_GEN_VERSION` in its `gen.h`;
  contest-template has no tags (`contest.py update tools` takes `main`).

## The skill

- `SKILL.md` is the entry point: modes → `workflows/*.md`, topics →
  `references/*.md`. Keep it short; details go to references.
- Written in English, for the agent: rules and the reason behind them, no
  marketing. Contestant-facing text (statements, editorials) is Polish.
- Seeds (gen.h ≥ 2.0): no `.seed` by default; a distinct `.seed(x)` for every
  random test only when the user wants tests that never change.

## Testing a change

Load the working copy instead of the installed plugin:
`claude --plugin-dir .`, or link it as a personal skill
(`ln -s "$PWD/skills/sinol-package" ~/.claude/skills/sinol-package`, and don't
have the plugin installed at the same time). Then try a real request in a
contest repository, e.g. a new package or a review.
