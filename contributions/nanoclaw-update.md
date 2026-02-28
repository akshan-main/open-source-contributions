# NanoClaw: Upstream Sync Skill

**PR**: [qwibitai/nanoclaw #217](https://github.com/qwibitai/nanoclaw/pull/217)
**Status**: Open (adopted by fork users; official version later shipped in [#372](https://github.com/qwibitai/nanoclaw/pull/372))

## Problem

NanoClaw (~16k stars) is a customizable AI agent platform where users fork and modify skills. When upstream ships fixes or new features, users with customized forks have no efficient way to sync. One user reported spending $15 in API credits trying to merge manually — Claude scans the full repo and refactors unrelated code during the process.

Issue [#181](https://github.com/qwibitai/nanoclaw/issues/181) ("Upgrade Skill") was opened as a feature request. Maintainer triaged it as valid.

## PR #217: Workflow playbook

A Claude Code skill (`.claude/skills/update-nanoclaw/SKILL.md`) that gives Claude a strict, low-token playbook for upstream sync:

- **Preflight**: checks clean working tree, sets up `upstream` remote, detects branch name
- **Backup**: timestamped branch + tag before any changes (`backup/pre-update-<hash>-<timestamp>`)
- **Preview**: `git log` + `git diff` against merge base, changes categorized by type (skills/source/config)
- **Update paths**: user picks merge (default), cherry-pick, rebase, or abort
- **Conflict preview**: dry-run merge (`git merge --no-commit --no-ff`) before committing
- **Resolution**: only opens conflicted files, never refactors surrounding code
- **Validation**: `npm run build` + `npm test`
- **Rollback**: backup tag printed at end of every run

Key design constraint: minimize token usage by using `git status`, `git log`, `git diff` for everything and only opening files with actual conflicts.

## Adoption

Maintainer gavrielc responded: "thanks for contributing! This meets a critical need. I will aim to review today and will also ask some people in discord to try on their clone."

Fork users adopted it before any official version existed:
- **timmoser/Shelby**: pushed commit referencing #217 — "improve upstream sync skill with safety net and validation"
- **TerrifiedBug/nanotars**: pushed commit referencing #217 — "skill: add /update-nanoclaw for upstream sync"

## PR #317: Script-driven architecture

Follow-up alternative with shell scripts instead of inline markdown:

- 9 scripts with deterministic exit codes and `KEY=VALUE` machine-readable output
- Test sandbox (`test-sandbox.sh`) creating isolated git environments
- 7 test scenarios: merge-conflict, clean-merge, no-upstream, bad-upstream, no-main-master, cherry-conflict, validate-fail
- CI workflow (`.github/workflows/update-nanoclaw-skill-tests.yml`)

## Note

The maintainer later shipped an official `/update` skill in [#372](https://github.com/qwibitai/nanoclaw/pull/372). #217 was the first implementation of this feature.

## Links

- PR #217 (playbook): https://github.com/qwibitai/nanoclaw/pull/217
- PR #317 (scripts): https://github.com/qwibitai/nanoclaw/pull/317
- Official version #372: https://github.com/qwibitai/nanoclaw/pull/372
- Feature request: https://github.com/qwibitai/nanoclaw/issues/181
