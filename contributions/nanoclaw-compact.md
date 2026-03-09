# NanoClaw: /compact Session Command

**PR**: [qwibitai/nanoclaw #817](https://github.com/qwibitai/nanoclaw/pull/817)
**Status**: Merged
**Reviewer**: gavrielc

## Problem

Long-running NanoClaw sessions accumulate context until the agent starts losing track of earlier conversation. The Claude Agent SDK has a built-in `/compact` slash command, but NanoClaw's architecture (orchestrator → container → SDK) means slash commands can't just be typed into a chat — the orchestrator intercepts all messages and wraps them in XML before forwarding.

[#322](https://github.com/qwibitai/nanoclaw/issues/322).

## What I built

A skill package that adds `/compact` as a session command. The implementation spans two layers:

**Orchestrator layer** (`session-commands.ts`):
- `extractSessionCommand()` parses `/compact` from messages (with or without trigger prefix), rejects partial matches and messages with extra text
- `isSessionCommandAllowed()` gates access — main group (self-chat) or device owner (`is_from_me`) only. Untrusted senders get a denial message if they're allowed to interact, or silent consumption if not
- `handleSessionCommand()` orchestrates the whole flow: processes pre-compact messages first so they're in the session context before compaction, then forwards `/compact` to a fresh container
- Pre-compact batching matters because if a user sends "summarize this" then "/compact" in the same polling interval, without handling the summary first it gets wiped by compaction

**Agent-runner layer** (container side):
- `KNOWN_SESSION_COMMANDS` whitelist prevents accidental interception of user messages that start with `/`
- Slash commands use `query({ prompt: "/compact" })` with a string prompt directly — not `MessageStream`, not array format. The SDK only recognizes slash commands at conversation start
- Strips `allowedTools` and `mcpServers` since compaction doesn't need tool access
- Tracks `compact_boundary` system event to confirm the SDK actually compacted, warns if not observed
- PreCompact hook archives the full transcript to `conversations/` before the SDK compacts it

The maintainer's review asked about reusing `runQuery` for slash commands and dropping pre-compact batching. I explained why both would break: `runQuery` uses `MessageStream` (array format) which the SDK doesn't recognize as a slash command, and dropping pre-compact batching causes data loss in the narrow window where messages arrive in the same polling batch as `/compact`.

He also suggested extracting session command interception from inline `index.ts` into a separate `handleSessionCommand()` in `session-commands.ts` — I did that in a follow-up commit, which cleaned up ~100 inline lines into a deps-interface pattern.

The infrastructure (`session-commands.ts`, `KNOWN_SESSION_COMMANDS`, interception points) is designed so future session commands just add their entry and implement their logic.

## Links

- PR #817 (merged): https://github.com/qwibitai/nanoclaw/pull/817
- Issue: https://github.com/qwibitai/nanoclaw/issues/322
