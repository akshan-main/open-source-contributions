# Open Source Contributions

PRs I've opened across various projects (that were succesful)

## Highlights

- **Merged**: [modelcontextprotocol/python-sdk #2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038) - progress notifications were silently dropped on streamable HTTP because `report_progress()` wasn't passing `related_request_id` to the transport layer. Bug had been open ~8 months.
- **Merged**: [ASML-Labs/dagster-delta #54](https://github.com/ASML-Labs/dagster-delta/pull/54) - deltalake 1.2.1+ changed how Arrow tables come back (`string_view` types, nondeterministic row order, shifted partition columns), which broke the whole test suite. Fixed assertions, bumped the dep, and also fixed the release workflow that was writing build artifacts to the wrong directory.
- **Led to merged fix**: [huggingface/trl #5064](https://github.com/huggingface/trl/pull/5064) - GRPO multimodal training was crashing because `prepare_multimodal_messages` was being called on already-templated string prompts. Found two more bugs in the same code path while I was in there. Maintainer's merged fix ([#5067](https://github.com/huggingface/trl/pull/5067)) links to this PR as its only context.
- **Merged**: [qwibitai/nanoclaw #217](https://github.com/qwibitai/nanoclaw/pull/217) - Claude Code skill for syncing customized forks with upstream. Preflight, backup, dry-run conflict preview, merge/cherry-pick/rebase/abort, validation. Fork users started using it before it was even reviewed. Maintainer replaced the old `/update` skill with it.
- **Merged**: [qwibitai/nanoclaw #705](https://github.com/qwibitai/nanoclaw/pull/705) - sender-based access control at the orchestrator level so denied senders never invoke the agent. Two modes (trigger: store messages but gate activation, drop: don't store at all), per-chat overrides. Originally submitted as a skill, maintainer asked for it in core instead.
- **Merged**: [qwibitai/nanoclaw #817](https://github.com/qwibitai/nanoclaw/pull/817) - `/compact` session command for manual context compaction. Auth-gated to main group or device owner, processes pre-compact messages first so context isn't lost, forwards the SDK's built-in `/compact` as a string prompt (not MessageStream). Two-layer implementation: orchestrator handles auth + batching, agent-runner handles SDK integration + compact boundary tracking.
- **Review adopted**: [pydantic/pydantic-ai #3772 review](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) - built tool approval for Vercel AI adapter independently, PR closed as dup. Reviewed the competing PR and pointed out it was copy-pasting the entire base class `dispatch_request`. Suggested `super()` delegation instead, which they adopted.

## Contributions

| Project | PR | Status | Summary | Detail |
|---|---|---|---|---|
| modelcontextprotocol/python-sdk | [#2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038) | Merged | Fix silent drop of progress notifications on streamable HTTP | [detail](contributions/mcp-python-sdk-progress.md) |
| ASML-Labs/dagster-delta | [#54](https://github.com/ASML-Labs/dagster-delta/pull/54) | Merged | deltalake >=1.2.1 compat + release workflow fix | [detail](contributions/dagster-delta-deltalake-compat.md) |
| huggingface/trl | [#5064](https://github.com/huggingface/trl/pull/5064) | Open (led to [#5067](https://github.com/huggingface/trl/pull/5067)) | Root-caused GRPO multimodal crash, found two more bugs in the same path | [detail](contributions/trl-grpo-multimodal-prompts.md) |
| huggingface/trl | [#5073](https://github.com/huggingface/trl/pull/5073) | Open | Fix vision encoder dtype crash when training VLMs with bf16/fp16 | [detail](contributions/trl-vlm-bf16-dtype.md) |
| pydantic/pydantic-ai | [#4283](https://github.com/pydantic/pydantic-ai/pull/4283) + [#3772 review](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) | Review adopted | Built tool approval for Vercel AI adapter, then reviewed competing PR and suggested arch fix that shipped | [detail](contributions/pydantic-ai-tool-approval.md) |
| qwibitai/nanoclaw | [#217](https://github.com/qwibitai/nanoclaw/pull/217) | Merged | Upstream sync skill for customized forks | [detail](contributions/nanoclaw-update.md) |
| qwibitai/nanoclaw | [#705](https://github.com/qwibitai/nanoclaw/pull/705) | Merged | Sender allowlist for per-chat access control, pre-agent filtering | [detail](contributions/nanoclaw-sender-allowlist.md) |
| qwibitai/nanoclaw | [#817](https://github.com/qwibitai/nanoclaw/pull/817) | Merged | /compact session command — auth-gated context compaction with pre-compact batching | [detail](contributions/nanoclaw-compact.md) |

## Notes

Each detail page has more context on what broke and how it was fixed.
