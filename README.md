# Open Source Contributions

PRs I've submitted to various open source projects — bug fixes, compatibility work, feature implementations.

## Highlights

- **Merged**: [modelcontextprotocol/python-sdk #2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038) — progress notifications were silently dropped on streamable HTTP because `related_request_id` wasn't being passed through. Bug had been open ~8 months.
- **Merged**: [ASML-Labs/dagster-delta #54](https://github.com/ASML-Labs/dagster-delta/pull/54) — deltalake 1.2.1+ broke the test suite in multiple ways (type changes, row ordering, partition columns). Also fixed their broken release workflow.
- **Led to merged fix**: [huggingface/trl #5064](https://github.com/huggingface/trl/pull/5064) — found the root cause behind 4 open issues with GRPO multimodal training. Maintainer's merged fix ([#5067](https://github.com/huggingface/trl/pull/5067)) links back to this PR as its only context.
- **Merged**: [qwibitai/nanoclaw #217](https://github.com/qwibitai/nanoclaw/pull/217) — upstream sync skill for customized forks. Fork users adopted it before it was even reviewed; maintainer replaced the old `/update` skill with it.
- **Review adopted**: [pydantic/pydantic-ai #3772 review](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) — built tool approval for Vercel AI adapter independently, then after my PR was closed as dup, reviewed the other PR and suggested a `super()` delegation pattern that they adopted.

## Contributions

| Project | PR | Status | Summary | Detail |
|---|---|---|---|---|
| modelcontextprotocol/python-sdk | [#2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038) | Merged | Fix progress notification routing on streamable HTTP transport | [detail](contributions/mcp-python-sdk-progress.md) |
| ASML-Labs/dagster-delta | [#54](https://github.com/ASML-Labs/dagster-delta/pull/54) | Merged | deltalake >=1.2.1 compat — test fixes, dep bump, release workflow fix | [detail](contributions/dagster-delta-deltalake-compat.md) |
| huggingface/trl | [#5064](https://github.com/huggingface/trl/pull/5064) | Open (led to [#5067](https://github.com/huggingface/trl/pull/5067)) | Root-caused GRPO multimodal crashes, maintainer's fix built on this | [detail](contributions/trl-grpo-multimodal-prompts.md) |
| huggingface/trl | [#5073](https://github.com/huggingface/trl/pull/5073) | Open | Fix bf16/fp16 dtype mismatch for VLM vision tensors in GRPO | [detail](contributions/trl-vlm-bf16-dtype.md) |
| pydantic/pydantic-ai | [#4283](https://github.com/pydantic/pydantic-ai/pull/4283) + [#3772 review](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) | Review adopted | Built tool approval feature, then reviewed competing PR — suggested arch change that shipped | [detail](contributions/pydantic-ai-tool-approval.md) |
| qwibitai/nanoclaw | [#217](https://github.com/qwibitai/nanoclaw/pull/217) | Merged | Upstream sync skill — replaced the old `/update` skill | [detail](contributions/nanoclaw-update.md) |

## Notes

Each detail page covers what broke, the fix, and relevant links.
