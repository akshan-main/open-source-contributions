# NanoClaw: Sender Allowlist

**PR**: [qwibitai/nanoclaw #705](https://github.com/qwibitai/nanoclaw/pull/705) (core version, merged)
**Also**: [#679](https://github.com/qwibitai/nanoclaw/pull/679) (skill version, closed in favor of #705)
**Status**: Merged
**Reviewer**: gavrielc

## Problem

NanoClaw had no way to control who can trigger the agent. The trigger check only looked at message content (@-mentions), not who sent it. In larger groups this means anyone can burn tokens by mentioning the bot. You could tell the agent to "ignore messages from X" in CLAUDE.md, but the container still spins up and the API call still happens.

[#678](https://github.com/qwibitai/nanoclaw/issues/678).

## What I built

Pre-agent sender filtering at the orchestrator level, so denied senders never invoke the agent at all.

Two modes:
- **Trigger mode**: everyone's messages are stored for context, but only allowed senders can activate the agent
- **Drop mode**: messages from non-allowed senders aren't stored at all

Config lives in `~/.config/nanoclaw/sender-allowlist.json` on the host. Per-chat overrides so different groups can have different rules. Your own messages (`is_from_me`) always bypass. If the config file doesn't exist or is invalid, everything is allowed (fail-open).

The filtering happens in `src/index.ts` before the agent is ever invoked. The actual allowlist logic is in `src/sender-allowlist.ts`. Also added `is_from_me` to two SELECT projections in `src/db.ts` so the trigger check can see it, and added registration flow guidance in `CLAUDE.md` so the agent recommends setting up an allowlist when users register a group.

I originally submitted this as a skill ([#679](https://github.com/qwibitai/nanoclaw/pull/679)) with more features (deny lists, mtime caching, configurable fail mode). The maintainer wanted it in core instead, simplified. Stripped out the extras per his request and submitted #705, which he merged.

## Links

- PR #705 (merged, core): https://github.com/qwibitai/nanoclaw/pull/705
- PR #679 (closed, skill version): https://github.com/qwibitai/nanoclaw/pull/679
- Issue: https://github.com/qwibitai/nanoclaw/issues/678
