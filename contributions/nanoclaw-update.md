# NanoClaw: Upstream Sync Skill

**PR**: [qwibitai/nanoclaw #217](https://github.com/qwibitai/nanoclaw/pull/217)
**Status**: Merged
**Reviewer**: gavrielc

## Problem

NanoClaw is a customizable AI agent platform where users fork and modify skills. When upstream ships updates, there's no good way to sync a customized fork. One user reported spending $15 in API credits trying to merge manually because Claude kept scanning the full repo and refactoring unrelated code.

Feature request: [#181](https://github.com/qwibitai/nanoclaw/issues/181).

## What I built

A Claude Code skill (`.claude/skills/update-nanoclaw/SKILL.md`) that gives Claude a strict playbook for upstream sync:

- Preflight checks (clean tree, upstream remote, branch detection)
- Backup branch + tag before any changes
- Preview via `git log` + `git diff` against merge base
- User picks merge (default), cherry-pick, rebase, or abort
- Dry-run merge before committing
- Only opens files with actual conflicts — no refactoring surrounding code
- Runs `npm run build` + `npm test` after
- Prints backup tag at the end for rollback

Main design goal: minimize token usage by sticking to git commands for everything and only opening files that actually have conflicts.

I also submitted [#317](https://github.com/qwibitai/nanoclaw/pull/317), a script-based alternative with shell scripts, test sandbox, and CI workflow. The maintainer preferred the simpler markdown-only approach and merged #217 instead — "we should keep it simple and only add scripts where it's really warranted."

## Adoption

A couple fork users adopted it before it was even reviewed:
- timmoser/Shelby pushed a commit referencing #217
- TerrifiedBug/nanotars pushed a commit referencing #217

After merging, the maintainer removed the old `/update` skill and replaced it with this one.

## Links

- PR #217 (merged): https://github.com/qwibitai/nanoclaw/pull/217
- PR #317 (closed, script alternative): https://github.com/qwibitai/nanoclaw/pull/317
- Feature request: https://github.com/qwibitai/nanoclaw/issues/181
