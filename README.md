# Open Source Contributions

Open-source work in ML infrastructure, inference performance, agent runtimes, and developer reliability. The work clusters around high-friction failure points: wasted GPU time, brittle training paths, leaky modular boundaries, unsafe agent triggers, expensive fork updates, and partial-failure behavior. The common thread is finding the right control point, whether that is a GPU sync in a hot path, image-vs-latent dimensions in video conditioning, message authorization before agent startup, or fork updates that should stay git-native instead of becoming a repo-wide model session.

## Quick Navigate

- [Selected Work](#selected-work) - strongest merged work and what it unlocks for users or maintainers.
- [Modular Diffusers](#modular-diffusers) - composable pipeline work in Hugging Face Diffusers.
- [Performance Engineering](#performance-engineering) - measured inference-path optimization.
- [Pipeline Correctness](#pipeline-correctness) - bug fixes in diffusion pipeline behavior.
- [Model Loading](#model-loading) - single-file checkpoint loading support.
- [Test Architecture Refactoring](#test-architecture-refactoring) - a maintainer-led diffusers test migration carried across 26 model test suites and twelve pipeline test files.
- [Agent Runtime](#agent-runtime) - authorization, session commands, and runtime inspection in NanoClaw.
- [Developer Tooling](#developer-tooling) - tools that make project maintenance and protocol behavior safer.
- [Training Reliability](#training-reliability) - TRL multimodal training failure analysis and focused fixes.
- [Architecture Review](#architecture-review) - design feedback that changed accepted implementations.
- [Bug Reports](#bug-reports) - reported bugs with repro and fix design.
- [Full Index](#full-index) - all entries in one table.

## Selected Work

### 1. QwenImage-Family Performance Fix

**PR**: [huggingface/diffusers #13406](https://github.com/huggingface/diffusers/pull/13406) (Merged)

**What changed**: I profiled the QwenImage transformer path in Perfetto, traced repeated RoPE frequency CPU-to-GPU transfers in the eager forward path, and replaced per-forward `.to(device)` calls with cached device frequencies via `lru_cache_unless_export` in both RoPE classes. The computation and outputs stay the same; the patch removes repeated transfer and synchronization work from the hot path.

**What it enables**: Default eager inference gets faster without requiring `torch.compile` or changing model behavior. The profile traced about `76ms` of `cudaStreamSynchronize` per `transformer_forward` to repeated RoPE device transfers. At 20 inference steps, that is roughly `~1.5s` less synchronization overhead. Because the optimized transformer path is shared, the fix applies across `QwenImage`, `QwenImageEdit`, `QwenImageEditPlus`, and `QwenImageLayered`.

Detail: [contributions/diffusers-qwenimage-rope-device-cache.md](contributions/diffusers-qwenimage-rope-device-cache.md)

### 2. NanoClaw Runtime Sender Gating

**PR**: [qwibitai/nanoclaw #705](https://github.com/qwibitai/nanoclaw/pull/705) (Merged)

**What changed**: I added sender allowlist enforcement before NanoClaw starts the agent: host config loading, per-chat rules, trigger/drop modes, owner bypass through `is_from_me`, DB projection updates, orchestrator checks, and focused tests.

**What it enables**: Shared-chat owners can separate “visible in context” from “allowed to trigger work.” Denied senders can be blocked before container startup, model invocation, token spend, and tool execution; stricter deployments can also drop denied messages before storage. The important part is the layer: this is enforced at the orchestrator boundary, not as a prompt instruction after the agent has already been invoked.

Detail: [contributions/nanoclaw-sender-allowlist.md](contributions/nanoclaw-sender-allowlist.md)

### 3. NanoClaw Low-Token Fork Updates

**PR**: [qwibitai/nanoclaw #217](https://github.com/qwibitai/nanoclaw/pull/217) (Merged)

**What changed**: I wrote `/update-nanoclaw`, a Claude Code skill for updating customized NanoClaw forks with clean-tree checks, upstream remote setup, backup branch/tag creation, upstream diff bucketing, dry-run conflict preview, merge/cherry-pick/rebase/abort choices, validation, and rollback instructions.

**What it enables**: Customized NanoClaw users can take upstream fixes without reinstalling, sacrificing local changes, or burning tokens on a model trying to reason across the whole repo. The skill keeps updates on a bounded git path: preview upstream drift, categorize changed files, dry-run conflicts, open only real conflict files, choose merge/cherry-pick/rebase intentionally, validate, and keep a rollback point. The maintainer called it a critical need.

Detail: [contributions/nanoclaw-update.md](contributions/nanoclaw-update.md)

### 4. LTX Video in Modular Diffusers

**PR**: [huggingface/diffusers #13378](https://github.com/huggingface/diffusers/pull/13378) (Merged)

**What changed**: I added the LTX Video modular pipeline in Diffusers: T2V and I2V block graphs, denoise-loop blocks, VAE/text encode-decode steps, pachifier support, `LTXAutoBlocks`, registry/export wiring, dependency dummies, and modular workflow tests.

**What it enables**: LTX users can work with the pipeline as inspectable stages instead of a single monolithic call: text encoding, image conditioning, latent preparation, denoising, decoding, and pachifying are exposed as blocks. That makes it practical to debug one stage, reuse loaded components, swap or extend only the part being researched, and route T2V/I2V through `LTXAutoBlocks` based on inputs without maintaining separate forked pipeline code.

Detail: [contributions/diffusers-modular-ltx-video-pipeline.md](contributions/diffusers-modular-ltx-video-pipeline.md)

### 5. NanoClaw `/compact` Session Command

**PR**: [qwibitai/nanoclaw #817](https://github.com/qwibitai/nanoclaw/pull/817) (Merged)

**What changed**: I added `/compact` as an auth-gated session command with command parsing, a reusable `handleSessionCommand()` path, pre-compact message batching, SDK-compatible raw slash-command execution, compact-boundary tracking, and transcript archival hook support.

**What it enables**: Users can manage long-running NanoClaw sessions from chat without losing the message that arrived right before compaction. Maintainers also get a reusable session-command path: commands are authorized, parsed, routed through the SDK form that actually mutates session state, and kept out of the normal message stream where they would be treated as plain text.

Detail: [contributions/nanoclaw-compact.md](contributions/nanoclaw-compact.md)

## Modular Diffusers

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| huggingface/diffusers | [#13378](https://github.com/huggingface/diffusers/pull/13378) | Added the LTX modular pipeline package with T2V/I2V block graphs, `LTXAutoBlocks`, registry/exports, dependency dummies, and tests | LTX users can inspect, run, replace, or extend individual pipeline stages instead of copying the whole video pipeline to customize one step | Merged | [detail](contributions/diffusers-modular-ltx-video-pipeline.md) |
| huggingface/diffusers | [#13389](https://github.com/huggingface/diffusers/pull/13389) | Added the HunyuanVideo 1.5 modular pipeline with T2V and I2V block graphs, registry/exports, dependency dummies, and tests; parity verified at MAD `0.000000` against both standard pipelines | HunyuanVideo 1.5 users can run T2V or I2V as inspectable stages and swap one stage without copying the whole video pipeline | Merged | [detail](contributions/diffusers-modular-hunyuanvideo15-pipeline.md) |
| huggingface/diffusers | [#13498](https://github.com/huggingface/diffusers/pull/13498) + [#13663](https://github.com/huggingface/diffusers/pull/13663) | Added the Ernie-Image modular pipeline (`ErnieImageAutoBlocks`), verified parity (MAD `0.000033`), then fixed review findings on prompt-enhancer skipping, VAE epsilon, and latent-output handling | Ernie-Image users get a stage-based pipeline that matches the standard one on conditional steps, normalization, and output contracts | Merged | [detail](contributions/diffusers-modular-ernie-image-pipeline.md) |

## Performance Engineering

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| huggingface/diffusers | [#13406](https://github.com/huggingface/diffusers/pull/13406) | Cached QwenImage RoPE freqs on device in the shared transformer path | Removes measured eager-mode synchronization stalls from a shared QwenImage-family hot path without output changes or requiring `torch.compile` | Merged | [detail](contributions/diffusers-qwenimage-rope-device-cache.md) |

## Pipeline Correctness

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| huggingface/diffusers | [#13440](https://github.com/huggingface/diffusers/pull/13440) | Renamed latent shape variables in HunyuanVideo 1.5 I2V so latent dimensions no longer overwrite requested pixel `height`/`width` | I2V users get conditioning based on the image resolution they requested, not a silent latent-size preprocessing path | Merged | [detail](contributions/diffusers-hunyuan15-i2v-pixel-resolution-fix.md) |
| huggingface/diffusers | [#13957](https://github.com/huggingface/diffusers/pull/13957) + [#13955](https://github.com/huggingface/diffusers/pull/13955) | Fixed the true-CFG gate in three flux pipelines to accept precomputed negative embeds, and closed three `check_inputs` gaps (embed shape check, operator-precedence bug, Redux scale validation) | Flux users get real guidance instead of silently none when passing precomputed negative embeds, and bad inputs fail fast with clear errors | Merged | [detail](contributions/diffusers-flux-true-cfg-validation.md) |
| huggingface/diffusers | [#13981](https://github.com/huggingface/diffusers/pull/13981) | Fixed four Bria FIBO crashes: `guidance_embeds` construction, unusable `prompt_embeds` args (removed), tensor image inputs, and malformed multi-image output | FIBO users can run guidance-embed checkpoints, pass tensor images, and get correctly shaped multi-image output | Merged | [detail](contributions/diffusers-bria-fibo-fixes.md) |

## Model Loading

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| huggingface/diffusers | [#13727](https://github.com/huggingface/diffusers/pull/13727) | Added `from_single_file` support to `ErnieImageTransformer2DModel`: loader mixin, registry entry, and a prefix-stripping checkpoint converter, verified by a save/reload round trip | Community single-file `.safetensors` ERNIE checkpoints load directly, the same as Flux and Chroma | Merged | [detail](contributions/diffusers-ernie-single-file.md) |

## Test Architecture Refactoring

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| huggingface/diffusers | [#13849](https://github.com/huggingface/diffusers/pull/13849), [#13845](https://github.com/huggingface/diffusers/pull/13845), [#13840](https://github.com/huggingface/diffusers/pull/13840), [#13835](https://github.com/huggingface/diffusers/pull/13835), [#13834](https://github.com/huggingface/diffusers/pull/13834), [#13826](https://github.com/huggingface/diffusers/pull/13826) | Migrated 12 model test suites (11 autoencoders plus the Sana transformer) from the old `unittest` classes to the shared config-plus-mixin structure, with seeded inputs and pytest assertions | Model tests follow one structure across suites, with each concern in its own class and deterministic inputs, while model-specific skips and `@slow` integration tests stay intact | Merged | [detail](contributions/diffusers-model-test-mixin-migration.md) |
| huggingface/diffusers | [#13832](https://github.com/huggingface/diffusers/pull/13832), [#13847](https://github.com/huggingface/diffusers/pull/13847), [#13891](https://github.com/huggingface/diffusers/pull/13891), [#13897](https://github.com/huggingface/diffusers/pull/13897), [#13898](https://github.com/huggingface/diffusers/pull/13898), [#13901](https://github.com/huggingface/diffusers/pull/13901) | Migrated 14 more model test suites (four video autoencoders, Cosmos ControlNet, and the UNet family) to the config-plus-mixin structure, with a bundled Mochi VAE `return_dict` fix | The UNet family and remaining video autoencoders read the same as the migrated suites, hub integration tests and expected slices stay intact, and a real API bug got fixed along the way | Merged | [detail](contributions/diffusers-unet-autoencoder-test-migration.md) |
| huggingface/diffusers | [#14220](https://github.com/huggingface/diffusers/pull/14220), [#14224](https://github.com/huggingface/diffusers/pull/14224), [#14228](https://github.com/huggingface/diffusers/pull/14228), [#14231](https://github.com/huggingface/diffusers/pull/14231), [#14235](https://github.com/huggingface/diffusers/pull/14235), [#14239](https://github.com/huggingface/diffusers/pull/14239), [#14240](https://github.com/huggingface/diffusers/pull/14240), [#14242](https://github.com/huggingface/diffusers/pull/14242), [#14276](https://github.com/huggingface/diffusers/pull/14276), [#14283](https://github.com/huggingface/diffusers/pull/14283), [#14284](https://github.com/huggingface/diffusers/pull/14284), [#14289](https://github.com/huggingface/diffusers/pull/14289) | Carried the new pipeline-test framework ([#14113](https://github.com/huggingface/diffusers/pull/14113)) across QwenImage plus the entire Wan and CogVideoX families: twelve test files in twelve PRs, the first community batches after the maintainer's Flux example | Pipeline tests share one config-plus-mixin structure with memory/offload coverage; the new coverage surfaced two real pre-existing bugs fixed upstream in [#14269](https://github.com/huggingface/diffusers/pull/14269) | Merged | [detail](contributions/diffusers-pipeline-test-mixin-migration.md) |

## Agent Runtime

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| qwibitai/nanoclaw | [#705](https://github.com/qwibitai/nanoclaw/pull/705) | Added sender allowlist enforcement before agent invocation, including trigger/drop modes, per-chat rules, owner bypass, DB projection changes, and tests | Shared-chat deployments can keep passive context while blocking untrusted senders before agent startup, token spend, and tool execution | Merged | [detail](contributions/nanoclaw-sender-allowlist.md) |
| qwibitai/nanoclaw | [#817](https://github.com/qwibitai/nanoclaw/pull/817) | Added reusable session-command handling for `/compact`, with auth checks, pre-compact batching, raw SDK slash-command execution, and compact-boundary tracking | Long-running chat sessions can be compacted safely, without losing same-poll messages or letting untrusted users disrupt active work | Merged | [detail](contributions/nanoclaw-compact.md) |
| qwibitai/nanoclaw | [#1086](https://github.com/qwibitai/nanoclaw/pull/1086) | Added read-only `/capabilities` and `/status` skills gated to the main channel | Operators can answer “what can this bot do?” and “is the runtime healthy?” from chat without granting write-capable diagnostics | Merged | [detail](contributions/nanoclaw-capabilities-status-skills.md) |

## Developer Tooling

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| qwibitai/nanoclaw | [#217](https://github.com/qwibitai/nanoclaw/pull/217) | Added `/update-nanoclaw`: a git-first update skill with backups, upstream diff preview, conflict dry-run, merge/cherry-pick/rebase choices, validation, and rollback | Customized fork users can take upstream fixes through a bounded merge workflow instead of spending tokens on broad, ad hoc repo surgery | Merged | [detail](contributions/nanoclaw-update.md) |
| modelcontextprotocol/python-sdk | [#2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038) | Threaded `Context.request_id` into `report_progress()` as `related_request_id` and added regression coverage | MCP clients can show progress for long-running streamable-HTTP tools on the correct request stream instead of dropping updates | Merged | [detail](contributions/mcp-python-sdk-progress.md) |
| ASML-Labs/dagster-delta | [#54](https://github.com/ASML-Labs/dagster-delta/pull/54) | Updated deltalake compatibility assertions for Arrow/schema/order changes and fixed release builds to write artifacts into `dist` | Maintainers can upgrade deltalake and publish releases without tests failing on storage representation details or missing build artifacts | Merged | [detail](contributions/dagster-delta-deltalake-compat.md) |

## Training Reliability

| Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| huggingface/trl | [#5064](https://github.com/huggingface/trl/pull/5064) | Traced multimodal GRPO failures to string prompts passed into image-message preparation, mixed-precision image tensors, and reward callback exception behavior | VLM training failures became actionable: maintainers could separate user prompt misuse from dtype handling and reward-function policy | Open; prompt guard landed in [#5067](https://github.com/huggingface/trl/pull/5067) | [detail](contributions/trl-grpo-multimodal-prompts.md) |
| huggingface/trl | [#5073](https://github.com/huggingface/trl/pull/5073) | Focused the dtype fix to cast only floating image tensors in the VLM GRPO path | Users training VLMs with bf16/fp16 avoid vision-path dtype crashes while integer metadata like `image_grid_thw` stays valid | Open | [detail](contributions/trl-vlm-bf16-dtype.md) |

## Architecture Review

| Project | Contribution | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| pydantic/pydantic-ai | [#4283](https://github.com/pydantic/pydantic-ai/pull/4283) + [#3772 review](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) | Built a duplicate Vercel tool-approval implementation, then suggested a smaller `run_stream_native()` / `super()` delegation pattern on the accepted PR | Adapter maintainers keep tool approval behavior without duplicating broad base-class dispatch logic that would drift over time | Review adopted | [detail](contributions/pydantic-ai-tool-approval.md) |

## Bug Reports

| Project | Issue | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|
| microsoft/PyRIT | [#2162](https://github.com/microsoft/PyRIT/issues/2162) | Reported `ImageResizingConverter` producing zero-sized dimensions on small images and accepting non-finite scale factors, with exact repro and fix design | Automated red-team image runs fail with clear validation errors instead of crashing mid-pipeline on an opaque Pillow error | Closed; fixed in [#2169](https://github.com/microsoft/PyRIT/pull/2169) based on my finding | [detail](contributions/pyrit-image-resizing-converter-bug.md) |

## Full Index

| Theme | Project | PR | What changed | User/maintainer value | Status | Detail |
|---|---|---|---|---|---|---|
| Modular Diffusers | huggingface/diffusers | [#13378](https://github.com/huggingface/diffusers/pull/13378) | LTX Video modular pipeline with T2V/I2V blocks, auto workflow routing, exports, and tests | Researchers can customize LTX at block boundaries, route T2V/I2V automatically, and avoid copying an entire video pipeline for one experiment | Merged | [detail](contributions/diffusers-modular-ltx-video-pipeline.md) |
| Modular Diffusers | huggingface/diffusers | [#13389](https://github.com/huggingface/diffusers/pull/13389) | HunyuanVideo 1.5 modular pipeline with T2V/I2V blocks, exports, and tests, parity verified at MAD `0.000000` | HunyuanVideo 1.5 users can run and customize T2V or I2V at block boundaries instead of copying the whole video pipeline | Merged | [detail](contributions/diffusers-modular-hunyuanvideo15-pipeline.md) |
| Modular Diffusers | huggingface/diffusers | [#13498](https://github.com/huggingface/diffusers/pull/13498) + [#13663](https://github.com/huggingface/diffusers/pull/13663) | Ernie-Image modular pipeline with verified parity, plus review-finding fixes to prompt-enhancer skipping, VAE epsilon, and latent output | Ernie-Image users get a stage-based pipeline that matches the standard one on conditional steps, normalization, and output contracts | Merged | [detail](contributions/diffusers-modular-ernie-image-pipeline.md) |
| Performance Engineering | huggingface/diffusers | [#13406](https://github.com/huggingface/diffusers/pull/13406) | QwenImage RoPE device cache in the shared transformer | QwenImage-family users avoid repeated CPU-to-GPU RoPE transfers in eager inference; maintainers get one behavior-preserving hot-path fix shared by all variants | Merged | [detail](contributions/diffusers-qwenimage-rope-device-cache.md) |
| Pipeline Correctness | huggingface/diffusers | [#13440](https://github.com/huggingface/diffusers/pull/13440) | HunyuanVideo 1.5 I2V latent-vs-pixel dimension fix | I2V conditioning respects the requested image size instead of silently using latent dimensions for image preprocessing | Merged | [detail](contributions/diffusers-hunyuan15-i2v-pixel-resolution-fix.md) |
| Pipeline Correctness | huggingface/diffusers | [#13957](https://github.com/huggingface/diffusers/pull/13957) + [#13955](https://github.com/huggingface/diffusers/pull/13955) | Flux true-CFG gate fix for precomputed negative embeds plus three `check_inputs` validation fixes | Precomputed negative embeds actually drive guidance, and bad inputs fail fast with clear errors instead of deep shape crashes or silent resizes | Merged | [detail](contributions/diffusers-flux-true-cfg-validation.md) |
| Pipeline Correctness | huggingface/diffusers | [#13981](https://github.com/huggingface/diffusers/pull/13981) | Four Bria FIBO crash fixes: guidance embeds, unusable prompt-embed args (removed), tensor images, multi-image output | Guidance-embed checkpoints construct, tensor images work, and multi-image output comes back correctly shaped | Merged | [detail](contributions/diffusers-bria-fibo-fixes.md) |
| Model Loading | huggingface/diffusers | [#13727](https://github.com/huggingface/diffusers/pull/13727) | `from_single_file` support for `ErnieImageTransformer2DModel` | Community single-file ERNIE checkpoints load directly, same as Flux/Chroma | Merged | [detail](contributions/diffusers-ernie-single-file.md) |
| Test Architecture Refactoring | huggingface/diffusers | [#13849](https://github.com/huggingface/diffusers/pull/13849), [#13845](https://github.com/huggingface/diffusers/pull/13845), [#13840](https://github.com/huggingface/diffusers/pull/13840), [#13835](https://github.com/huggingface/diffusers/pull/13835), [#13834](https://github.com/huggingface/diffusers/pull/13834), [#13826](https://github.com/huggingface/diffusers/pull/13826) | 12 model test suites migrated to the shared config-plus-mixin structure across six PRs | Model tests follow one structure with each concern in its own class and deterministic seeded inputs, while model-specific skips and `@slow` tests stay intact | Merged | [detail](contributions/diffusers-model-test-mixin-migration.md) |
| Test Architecture Refactoring | huggingface/diffusers | [#13832](https://github.com/huggingface/diffusers/pull/13832), [#13847](https://github.com/huggingface/diffusers/pull/13847), [#13891](https://github.com/huggingface/diffusers/pull/13891), [#13897](https://github.com/huggingface/diffusers/pull/13897), [#13898](https://github.com/huggingface/diffusers/pull/13898), [#13901](https://github.com/huggingface/diffusers/pull/13901) | 14 more model test suites (UNet family, Cosmos ControlNet, four video autoencoders) migrated, with a bundled Mochi VAE `return_dict` fix | The migration now covers the UNet family and remaining video autoencoders with hub tests and expected slices intact | Merged | [detail](contributions/diffusers-unet-autoencoder-test-migration.md) |
| Test Architecture Refactoring | huggingface/diffusers | [#14220](https://github.com/huggingface/diffusers/pull/14220), [#14224](https://github.com/huggingface/diffusers/pull/14224), [#14228](https://github.com/huggingface/diffusers/pull/14228), [#14231](https://github.com/huggingface/diffusers/pull/14231), [#14235](https://github.com/huggingface/diffusers/pull/14235), [#14239](https://github.com/huggingface/diffusers/pull/14239), [#14240](https://github.com/huggingface/diffusers/pull/14240), [#14242](https://github.com/huggingface/diffusers/pull/14242), [#14276](https://github.com/huggingface/diffusers/pull/14276), [#14283](https://github.com/huggingface/diffusers/pull/14283), [#14284](https://github.com/huggingface/diffusers/pull/14284), [#14289](https://github.com/huggingface/diffusers/pull/14289) | Pipeline-test framework ([#14113](https://github.com/huggingface/diffusers/pull/14113)) carried across QwenImage and the Wan and CogVideoX families, twelve files in twelve PRs | Pipeline tests share one structure with memory/offload coverage; the new coverage surfaced two real pre-existing bugs fixed in [#14269](https://github.com/huggingface/diffusers/pull/14269) | Merged | [detail](contributions/diffusers-pipeline-test-mixin-migration.md) |
| Agent Runtime | qwibitai/nanoclaw | [#705](https://github.com/qwibitai/nanoclaw/pull/705) | Sender allowlist before agent invocation | Group owners can separate “visible in context” from “allowed to trigger work,” blocking unwanted activations before inference starts | Merged | [detail](contributions/nanoclaw-sender-allowlist.md) |
| Agent Runtime | qwibitai/nanoclaw | [#817](https://github.com/qwibitai/nanoclaw/pull/817) | Reusable `/compact` session-command path | Users can compact long sessions safely from chat; maintainers get a clean base for future session commands | Merged | [detail](contributions/nanoclaw-compact.md) |
| Agent Runtime | qwibitai/nanoclaw | [#1086](https://github.com/qwibitai/nanoclaw/pull/1086) | Read-only `/capabilities` and `/status` skills | Operators can diagnose runtime capability and health without handing the agent a write-capable instruction | Merged | [detail](contributions/nanoclaw-capabilities-status-skills.md) |
| Developer Tooling | qwibitai/nanoclaw | [#217](https://github.com/qwibitai/nanoclaw/pull/217) | Git-native `/update-nanoclaw` fork-update skill | Customized fork users can take upstream fixes through previewed diffs, real conflict files, validation, and rollback instead of repo-wide model guessing | Merged | [detail](contributions/nanoclaw-update.md) |
| Developer Tooling | modelcontextprotocol/python-sdk | [#2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038) | `related_request_id` progress routing | MCP clients can show progress for long-running tools on the correct streamable-HTTP request | Merged | [detail](contributions/mcp-python-sdk-progress.md) |
| Developer Tooling | ASML-Labs/dagster-delta | [#54](https://github.com/ASML-Labs/dagster-delta/pull/54) | deltalake compatibility fixes plus release artifact output path | Maintainers can upgrade storage dependencies and publish releases without brittle schema/order assertions blocking them | Merged | [detail](contributions/dagster-delta-deltalake-compat.md) |
| Training Reliability | huggingface/trl | [#5064](https://github.com/huggingface/trl/pull/5064) | GRPO multimodal crash analysis across prompt format, dtype, and reward callback paths | VLM training bugs became separable fixes instead of a vague “GRPO is broken” report | Open; prompt guard landed in [#5067](https://github.com/huggingface/trl/pull/5067) | [detail](contributions/trl-grpo-multimodal-prompts.md) |
| Training Reliability | huggingface/trl | [#5073](https://github.com/huggingface/trl/pull/5073) | VLM image tensor dtype handling | Mixed-precision VLM training can cast image tensors correctly without corrupting integer metadata | Open | [detail](contributions/trl-vlm-bf16-dtype.md) |
| Architecture Review | pydantic/pydantic-ai | [#4283](https://github.com/pydantic/pydantic-ai/pull/4283) + [#3772 review](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) | Tool-approval adapter review with `super()` delegation recommendation | Protocol adapter code stays closer to the base class, reducing future drift while keeping tool approval behavior | Review adopted | [detail](contributions/pydantic-ai-tool-approval.md) |
| Bug Reports | microsoft/PyRIT | [#2162](https://github.com/microsoft/PyRIT/issues/2162) | Zero-dimension and non-finite scale-factor holes in `ImageResizingConverter`, with repro and fix design | Batch image-attack runs fail with clear validation errors instead of an opaque mid-pipeline Pillow crash | Closed; fixed in [#2169](https://github.com/microsoft/PyRIT/pull/2169) based on my finding | [detail](contributions/pyrit-image-resizing-converter-bug.md) |
