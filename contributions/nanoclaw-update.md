# NanoClaw: Upstream Sync Skill

**PR**: [qwibitai/nanoclaw #217](https://github.com/qwibitai/nanoclaw/pull/217)
**Status**: Merged
**Reviewer**: gavrielc

## Problem

NanoClaw is a customizable AI agent platform where users fork and modify skills. When upstream ships updates, there's no good way to sync a customized fork. One user reported spending $15 in API credits trying to merge manually because Claude kept scanning the full repo and refactoring unrelated code.

Feature request: [#181](https://github.com/qwibitai/nanoclaw/issues/181).

## What I built

A Claude Code skill that gives Claude a strict playbook for upstream sync instead of letting it freelance. The whole point is to keep token usage low by using git commands for everything and only opening files that actually have conflicts.

The flow: check for clean working tree and set up the upstream remote, create a timestamped backup branch + tag, show what upstream changed (grouped by skills/source/config), let the user pick merge/cherry-pick/rebase/abort, do a dry-run merge to preview conflicts before committing, resolve only conflicted files (no refactoring surrounding code), run build + tests, print rollback instructions.

If rebase hits more than 3 rounds of conflicts it aborts and recommends merge instead. If the build fails after merging, it only tries to fix things clearly caused by the merge, not random stuff.

I also submitted [#317](https://github.com/qwibitai/nanoclaw/pull/317), a script-based alternative with shell scripts, a test sandbox, and CI. The maintainer preferred the simpler markdown-only approach: "we should keep it simple and only add scripts where it's really warranted." Merged #217, closed #317.

## Adoption

Fork users started using it before it was even reviewed:
- timmoser/Shelby pushed a commit referencing #217
- TerrifiedBug/nanotars pushed a commit referencing #217

After merging, the maintainer removed the old `/update` skill and replaced it with this one. Other forks (index-engine/nanoclaw) picked it up within hours.

## Links

- PR #217 (merged): https://github.com/qwibitai/nanoclaw/pull/217
- PR #317 (closed, script alternative): https://github.com/qwibitai/nanoclaw/pull/317
- Feature request: https://github.com/qwibitai/nanoclaw/issues/181
