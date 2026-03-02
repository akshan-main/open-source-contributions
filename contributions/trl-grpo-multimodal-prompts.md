# TRL: GRPO Multimodal Prompt Crash

**PR**: [huggingface/trl #5064](https://github.com/huggingface/trl/pull/5064)
**Status**: Open (maintainer's fix [#5067](https://github.com/huggingface/trl/pull/5067) merged, built on this)

## The bug

GRPO multimodal training crashes with `TypeError: string indices must be integers, not 'str'` when prompts are strings (what you get from `processor.apply_chat_template`). This is exactly what happens if you follow the official example notebook `grpo_qwen3_vl.ipynb`. Reported across [#4746](https://github.com/huggingface/trl/issues/4746), [#4870](https://github.com/huggingface/trl/issues/4870), [#4451](https://github.com/huggingface/trl/issues/4451), [#5041](https://github.com/huggingface/trl/issues/5041).

## What I found

All 4 issues traced back to the same thing: when images are present, `_generate_and_score_completions` unconditionally passes prompts through `prepare_multimodal_messages()`, which expects `{"role": ..., "content": ...}` dicts. When prompts are already-templated strings, `prompt["role"]` indexes into a string character and crashes.

While tracing through that code path I found two more bugs:

**Dtype mismatch** - the processor outputs float32 tensors (`pixel_values` etc) but when training with `bf16=True` the vision encoder expects bfloat16. The existing `_prepare_inputs` only handles DeepSpeed casting, not the general case. Crashes in `torch.layer_norm`. Added dtype detection from training args and cast only floating-point tensors (integer tensors like `image_grid_thw` left alone).

**Reward function crash propagation** - any exception in a reward function killed training entirely. Added try/except with NaN fallback for both sync and async paths, matching the `None->NaN` pattern already in the codebase.

Wrote tests for all three fixes.

## What happened

The maintainer agreed with the root cause but preferred a different approach for the string prompt fix: raise a clear `ValueError` telling users not to pre-apply chat templates, rather than silently handling strings. He opened [#5067](https://github.com/huggingface/trl/pull/5067) which adds that check across 4 trainers and fixes the example notebook. That PR links to #5064 as its only context.

He asked me to split out the dtype fix into [#5073](https://github.com/huggingface/trl/pull/5073).

On reward exception handling, his take was that belongs in userland, not the trainer.

## Links

- My PR: https://github.com/huggingface/trl/pull/5064
- Maintainer's merged fix: https://github.com/huggingface/trl/pull/5067
- Follow-up dtype fix: https://github.com/huggingface/trl/pull/5073
- Issues: [#4746](https://github.com/huggingface/trl/issues/4746), [#4870](https://github.com/huggingface/trl/issues/4870), [#4451](https://github.com/huggingface/trl/issues/4451), [#5041](https://github.com/huggingface/trl/issues/5041)
