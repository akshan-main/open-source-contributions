# NanoClaw: Sender Allowlist Before Agent Invocation

**PR**: [qwibitai/nanoclaw #705](https://github.com/qwibitai/nanoclaw/pull/705)
**Status**: Merged

## Change

- Added orchestrator-level sender allowlist checks before agent invocation.
- Added two runtime modes:
  - **Trigger mode**: keep messages for context, but allow only approved senders to trigger runs.
  - **Drop mode**: reject non-approved senders before their messages are stored.
- Added per-chat overrides through host config.
- Threaded `is_from_me` through DB query paths so owner messages can bypass allowlist checks.
- Added focused test coverage for allowlist behavior.

## What it enables

- Group owners can run NanoClaw in shared chats without letting every participant trigger paid model work.
- Trigger mode supports “visible in context but not allowed to wake the bot,” which is useful for group conversations where not every sender should have operator power.
- Drop mode supports stricter deployments where denied messages should not be stored at all.
- The security boundary is enforced before container startup, model invocation, token spend, and tool execution, not in a prompt after the agent has already launched.
- This matters architecturally because the policy is enforced at the deterministic orchestrator layer rather than relying on the model to obey an instruction after the expensive work has already started.

## Code notes

- `src/sender-allowlist.ts`: config parsing, validation, mode selection, and allow checks.
- `src/index.ts`: enforcement in trigger detection and message storage.
- `src/db.ts`: `is_from_me` added to message projections used by the trigger path.
- `src/sender-allowlist.test.ts`: behavior coverage for defaults, invalid config, trigger mode, drop mode, per-chat overrides, and owner bypass.

## Links

- PR: https://github.com/qwibitai/nanoclaw/pull/705
- Earlier approach: https://github.com/qwibitai/nanoclaw/pull/679
- Issue: https://github.com/qwibitai/nanoclaw/issues/678
