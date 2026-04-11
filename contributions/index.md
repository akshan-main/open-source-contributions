# Contributions Index

## Sections

- [Modular Diffusers](#modular-diffusers)
- [Performance Engineering](#performance-engineering)
- [Video Pipeline Correctness](#video-pipeline-correctness)
- [Agent Runtime](#agent-runtime)
- [Developer Tooling](#developer-tooling)
- [Training Reliability](#training-reliability)
- [Architecture Review](#architecture-review)
- [Full Table](#full-table)

## Modular Diffusers

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| huggingface/diffusers | [#13378](https://github.com/huggingface/diffusers/pull/13378) | Added the LTX modular pipeline package with T2V/I2V block graphs, `LTXAutoBlocks`, registry/exports, dependency dummies, and tests | LTX users can inspect, run, replace, or extend individual pipeline stages instead of copying the whole video pipeline to customize one step | Merged | [diffusers-modular-ltx-video-pipeline.md](diffusers-modular-ltx-video-pipeline.md) |

## Performance Engineering

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| huggingface/diffusers | [#13406](https://github.com/huggingface/diffusers/pull/13406) | Cached QwenImage RoPE freqs on device in the shared transformer path | Faster eager QwenImage-family generation without output changes or requiring users to rely on `torch.compile` to hide the sync | Merged | [diffusers-qwenimage-rope-device-cache.md](diffusers-qwenimage-rope-device-cache.md) |

## Video Pipeline Correctness

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| huggingface/diffusers | [#13440](https://github.com/huggingface/diffusers/pull/13440) | Renamed latent shape variables in HunyuanVideo 1.5 I2V so latent dimensions no longer overwrite requested pixel `height`/`width` | I2V users get conditioning based on the image resolution they requested, not a silent latent-size preprocessing path | Merged | [diffusers-hunyuan15-i2v-pixel-resolution-fix.md](diffusers-hunyuan15-i2v-pixel-resolution-fix.md) |

## Agent Runtime

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| qwibitai/nanoclaw | [#705](https://github.com/qwibitai/nanoclaw/pull/705) | Added sender allowlist enforcement before agent invocation, including trigger/drop modes, per-chat rules, owner bypass, DB projection changes, and tests | Shared-chat deployments can keep context from passive members while limiting who can trigger paid agent work | Merged | [nanoclaw-sender-allowlist.md](nanoclaw-sender-allowlist.md) |
| qwibitai/nanoclaw | [#817](https://github.com/qwibitai/nanoclaw/pull/817) | Added reusable session-command handling for `/compact`, with auth checks, pre-compact batching, raw SDK slash-command execution, and compact-boundary tracking | Long-running chat sessions can be compacted safely, without losing same-poll messages or letting untrusted users disrupt active work | Merged | [nanoclaw-compact.md](nanoclaw-compact.md) |
| qwibitai/nanoclaw | [#1086](https://github.com/qwibitai/nanoclaw/pull/1086) | Added read-only `/capabilities` and `/status` skills gated to the main channel | Operators can answer “what can this bot do?” and “is the runtime healthy?” from chat without granting write-capable diagnostics | Merged | [nanoclaw-capabilities-status-skills.md](nanoclaw-capabilities-status-skills.md) |

## Developer Tooling

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| qwibitai/nanoclaw | [#217](https://github.com/qwibitai/nanoclaw/pull/217) | Added `/update-nanoclaw`: a git-first update skill with backups, upstream diff preview, conflict dry-run, merge/cherry-pick/rebase choices, validation, and rollback | Fork users can take upstream fixes repeatedly while keeping local customizations recoverable and avoiding repo-wide token burn | Merged | [nanoclaw-update.md](nanoclaw-update.md) |
| modelcontextprotocol/python-sdk | [#2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038) | Threaded `Context.request_id` into `report_progress()` as `related_request_id` and added regression coverage | MCP clients can show progress for long-running streamable-HTTP tools on the correct request stream instead of dropping updates | Merged | [mcp-python-sdk-progress.md](mcp-python-sdk-progress.md) |
| ASML-Labs/dagster-delta | [#54](https://github.com/ASML-Labs/dagster-delta/pull/54) | Updated deltalake compatibility assertions for Arrow/schema/order changes and fixed release builds to write artifacts into `dist` | Maintainers can upgrade deltalake and publish releases without tests failing on storage representation details or missing build artifacts | Merged | [dagster-delta-deltalake-compat.md](dagster-delta-deltalake-compat.md) |

## Training Reliability

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| huggingface/trl | [#5064](https://github.com/huggingface/trl/pull/5064) | Traced multimodal GRPO failures to string prompts passed into image-message preparation, mixed-precision image tensors, and reward callback exception behavior | VLM training failures became actionable: maintainers could separate user prompt misuse from dtype handling and reward-function policy | Open; prompt guard landed in [#5067](https://github.com/huggingface/trl/pull/5067) | [trl-grpo-multimodal-prompts.md](trl-grpo-multimodal-prompts.md) |
| huggingface/trl | [#5073](https://github.com/huggingface/trl/pull/5073) | Focused the dtype fix to cast only floating image tensors in the VLM GRPO path | Users training VLMs with bf16/fp16 avoid vision-path dtype crashes while integer metadata like `image_grid_thw` stays valid | Open | [trl-vlm-bf16-dtype.md](trl-vlm-bf16-dtype.md) |

## Architecture Review

| Project | PR / Review | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| pydantic/pydantic-ai | [#4283](https://github.com/pydantic/pydantic-ai/pull/4283) + [#3772 review](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) | Built a duplicate Vercel tool-approval implementation, then suggested a smaller `run_stream_native()` / `super()` delegation pattern on the accepted PR | Adapter maintainers keep tool approval behavior without duplicating broad base-class dispatch logic that would drift over time | Review adopted | [pydantic-ai-tool-approval.md](pydantic-ai-tool-approval.md) |

## Full Table

| Theme | Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|---|
| Modular Diffusers | huggingface/diffusers | [#13378](https://github.com/huggingface/diffusers/pull/13378) | LTX Video modular pipeline with T2V/I2V blocks, auto workflow routing, exports, and tests | Researchers can customize LTX at block boundaries, route T2V/I2V automatically, and avoid copying an entire video pipeline for one experiment | Merged | [diffusers-modular-ltx-video-pipeline.md](diffusers-modular-ltx-video-pipeline.md) |
| Performance Engineering | huggingface/diffusers | [#13406](https://github.com/huggingface/diffusers/pull/13406) | QwenImage RoPE device cache in the shared transformer | QwenImage-family users get faster eager inference without behavior changes; maintainers get one hot-path fix shared by all variants | Merged | [diffusers-qwenimage-rope-device-cache.md](diffusers-qwenimage-rope-device-cache.md) |
| Video Pipeline Correctness | huggingface/diffusers | [#13440](https://github.com/huggingface/diffusers/pull/13440) | HunyuanVideo 1.5 I2V latent-vs-pixel dimension fix | I2V conditioning respects the requested image size instead of silently using latent dimensions for image preprocessing | Merged | [diffusers-hunyuan15-i2v-pixel-resolution-fix.md](diffusers-hunyuan15-i2v-pixel-resolution-fix.md) |
| Agent Runtime | qwibitai/nanoclaw | [#705](https://github.com/qwibitai/nanoclaw/pull/705) | Sender allowlist before agent invocation | Group owners can separate “visible in context” from “allowed to trigger work,” protecting token spend in shared chats | Merged | [nanoclaw-sender-allowlist.md](nanoclaw-sender-allowlist.md) |
| Agent Runtime | qwibitai/nanoclaw | [#817](https://github.com/qwibitai/nanoclaw/pull/817) | Reusable `/compact` session-command path | Users can compact long sessions safely from chat; maintainers get a clean base for future session commands | Merged | [nanoclaw-compact.md](nanoclaw-compact.md) |
| Agent Runtime | qwibitai/nanoclaw | [#1086](https://github.com/qwibitai/nanoclaw/pull/1086) | Read-only `/capabilities` and `/status` skills | Operators can diagnose runtime capability and health without handing the agent a write-capable instruction | Merged | [nanoclaw-capabilities-status-skills.md](nanoclaw-capabilities-status-skills.md) |
| Developer Tooling | qwibitai/nanoclaw | [#217](https://github.com/qwibitai/nanoclaw/pull/217) | Git-native `/update-nanoclaw` fork-update skill | Customized fork users can take upstream fixes with bounded conflict resolution, validation, and rollback | Merged | [nanoclaw-update.md](nanoclaw-update.md) |
| Developer Tooling | modelcontextprotocol/python-sdk | [#2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038) | `related_request_id` progress routing | MCP clients can show progress for long-running tools on the correct streamable-HTTP request | Merged | [mcp-python-sdk-progress.md](mcp-python-sdk-progress.md) |
| Developer Tooling | ASML-Labs/dagster-delta | [#54](https://github.com/ASML-Labs/dagster-delta/pull/54) | deltalake compatibility fixes plus release artifact output path | Maintainers can upgrade storage dependencies and publish releases without brittle schema/order assertions blocking them | Merged | [dagster-delta-deltalake-compat.md](dagster-delta-deltalake-compat.md) |
| Training Reliability | huggingface/trl | [#5064](https://github.com/huggingface/trl/pull/5064) | GRPO multimodal crash analysis across prompt format, dtype, and reward callback paths | VLM training bugs became separable fixes instead of a vague “GRPO is broken” report | Open; prompt guard landed in [#5067](https://github.com/huggingface/trl/pull/5067) | [trl-grpo-multimodal-prompts.md](trl-grpo-multimodal-prompts.md) |
| Training Reliability | huggingface/trl | [#5073](https://github.com/huggingface/trl/pull/5073) | VLM image tensor dtype handling | Mixed-precision VLM training can cast image tensors correctly without corrupting integer metadata | Open | [trl-vlm-bf16-dtype.md](trl-vlm-bf16-dtype.md) |
| Architecture Review | pydantic/pydantic-ai | [#4283](https://github.com/pydantic/pydantic-ai/pull/4283) + [#3772 review](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) | Tool-approval adapter review with `super()` delegation recommendation | Protocol adapter code stays closer to the base class, reducing future drift while keeping tool approval behavior | Review adopted | [pydantic-ai-tool-approval.md](pydantic-ai-tool-approval.md) |
