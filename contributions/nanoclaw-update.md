# NanoClaw: Low-Token Updates for Customized Forks

**PR**: [qwibitai/nanoclaw #217](https://github.com/qwibitai/nanoclaw/pull/217)
**Status**: Merged

## Change

- Added `/update-nanoclaw`, a Claude Code skill for updating customized NanoClaw forks.
- Required a clean worktree before any update work starts.
- Added upstream remote discovery/setup and branch detection.
- Created a rollback point with both backup branch and tag before touching user code.
- Previewed upstream commits and changed files from the merge base.
- Bucketed upstream changes into skills, source, build/config, and other files so risk is visible before merging.
- Offered merge, cherry-pick, rebase, or abort instead of forcing one update strategy.
- Added dry-run conflict preview before the real merge path.
- Instructed Claude to open only conflicted files and avoid unrelated refactors.
- Ended with build/test validation and explicit rollback instructions.

## What it enables

- Users with customized NanoClaw installs can keep taking upstream fixes without reinstalling or sacrificing local changes.
- The failure mode was real: users could spend significant API credits while Claude scanned the whole project, guessed at conflicts, and rewrote unrelated code.
- The skill turns updates into a bounded git workflow: inspect diffs, choose an update path, resolve only actual conflicts, validate, and keep a rollback handle.
- Maintainers get a supportable update path for forked installs instead of repeatedly debugging ad hoc manual merges.
- The design treats the hard part as operational safety, not just git syntax: reduce the files the model opens, preserve user changes, and make rollback available before anything risky happens.
- The maintainer called it a critical need, which matches the problem: customized installs need updates without losing local changes.

## Code notes

The skill is instruction-driven and intentionally git-first:

- `git status --porcelain` stops dirty-tree updates.
- `git merge-base`, `git log`, and `git diff --name-only` explain upstream drift without opening files.
- Backup branch and tag are created before merge/rebase/cherry-pick work.
- Dry-run merge lists conflicts before committing to the update path.
- Conflict resolution is scoped to conflict markers, not surrounding refactors.
- Rollback is a concrete `git reset --hard <backup-tag>` path.

## Links

- PR: https://github.com/qwibitai/nanoclaw/pull/217
- Alternate script-based proposal: https://github.com/qwibitai/nanoclaw/pull/317
- Feature request: https://github.com/qwibitai/nanoclaw/issues/181
