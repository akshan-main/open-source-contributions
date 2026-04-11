# NanoClaw: `/compact` Session Command

**PR**: [qwibitai/nanoclaw #817](https://github.com/qwibitai/nanoclaw/pull/817)
**Status**: Merged

## Change

- Added `/compact` as an auth-gated session command.
- Extracted command parsing, authorization, cursor handling, and pre-compact processing into `session-commands.ts`.
- Routed `/compact` through the SDK-recognized raw string prompt path instead of normal formatted chat messages.
- Added pre-compact batching so messages that arrive before `/compact` in the same poll are committed to session context first.
- Added compact-boundary tracking to detect whether compaction actually completed.
- Added transcript archival hook support before compaction.

## What it enables

- Users can keep long NanoClaw sessions usable from chat instead of needing an out-of-band reset or manual intervention.
- `/compact` changes session state; sending it through the normal message stream can leave it as literal text instead of an SDK slash command.
- Same-poll batching prevents a normal message immediately before `/compact` from being erased from useful context.
- Unauthorized users cannot use `/compact` to disrupt active work.
- Maintainers get a reusable command path for future session commands such as `/clear`, without piling more logic into the main orchestrator file.

## Code notes

### Orchestrator side

- `extractSessionCommand()` recognizes supported session commands after trigger-prefix stripping.
- `isSessionCommandAllowed()` gates commands to trusted contexts.
- `handleSessionCommand()` handles auth, same-poll batching, command execution, cursor movement, and denial behavior through injected dependencies.

### Agent-runner side

- Known session commands are intercepted before the normal query loop.
- `/compact` is sent as `query({ prompt: "/compact" })`, because the SDK only recognizes it as a slash command in that form.
- Tool fields are stripped from compaction-only runs.
- `compact_boundary` is observed so completion is not inferred blindly.

## Links

- PR: https://github.com/qwibitai/nanoclaw/pull/817
- Issue: https://github.com/qwibitai/nanoclaw/issues/322
