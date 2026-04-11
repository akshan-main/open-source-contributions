# NanoClaw: `/capabilities` and `/status` Skills

**PR**: [qwibitai/nanoclaw #1086](https://github.com/qwibitai/nanoclaw/pull/1086)
**Status**: Merged

## Change

- Added two read-only container skills:
  - `container/skills/capabilities/SKILL.md`
  - `container/skills/status/SKILL.md`
- Gated both commands to the main channel by checking for the `/workspace/project` mount.
- Added inspection paths for installed skills, tool families, MCP tool surface, container utilities, workspace mounts, IPC, and scheduled tasks.

## What it enables

- Operators can ask the bot what it can access and what is installed before trusting it with a task.
- `/capabilities` answers capability questions: installed skills, available tools, MCP actions, and container utilities.
- `/status` answers health questions: current context, workspace visibility, IPC, runtime tools, and scheduled tasks.
- Both commands are read-only, so diagnostics do not require giving the agent a write-oriented instruction.
- Main-channel gating avoids exposing system-level inspection from secondary chats that do not have full context.

## Code notes

### `/capabilities`

- Lists installed skills under `/home/node/.claude/skills/`.
- Reports allowed tool families and NanoClaw MCP commands.
- Checks container utility availability.
- Produces a structured capability report based on what is actually installed.

### `/status`

- Reports session context and working directory.
- Checks workspace, group, extra mount, and IPC visibility.
- Verifies key utility availability and versions.
- Uses MCP task listing for scheduled-task state.

Both skills stop immediately outside the main channel.

## Links

- PR: https://github.com/qwibitai/nanoclaw/pull/1086
