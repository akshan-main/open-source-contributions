# TRL: GRPO Multimodal Prompt Crash

**PR**: [huggingface/trl #5064](https://github.com/huggingface/trl/pull/5064)
**Status**: Open (maintainer's fix [#5067](https://github.com/huggingface/trl/pull/5067) merged, built on this)

## The bug

GRPO multimodal training crashes with `TypeError: string indices must be integers, not 'str'` when prompts are strings (the output of `processor.apply_chat_template`). This is what you get if you follow the official example notebook `grpo_qwen3_vl.ipynb`. Reported across issues [#4746](https://github.com/huggingface/trl/issues/4746), [#4870](https://github.com/huggingface/trl/issues/4870), [#4451](https://github.com/huggingface/trl/issues/4451), [#5041](https://github.com/huggingface/trl/issues/5041).

## What I found

All 4 issues traced back to the same thing: `_generate_and_score_completions` unconditionally passes prompts to `prepare_multimodal_messages()` when images are present, but that function expects `{"role": ..., "content": ...}` dicts. When prompts are strings, indexing into a string with `["role"]` blows up.

While digging through that code path I found two more problems:

1. **pixel_values dtype mismatch** — `forward_kwargs` stay float32 after `_prepare_inputs` but vision encoder weights are bf16/fp16. Crashes in `torch.layer_norm`.
2. **Reward function exceptions** — any exception in reward functions kills training entirely with no useful error.

Wrote fixes and tests for all three.

## What happened

The maintainer agreed with the root cause but preferred a different fix for the string prompt issue — instead of silently handling strings, raise a clear `ValueError` telling users not to pre-apply chat templates. He opened [#5067](https://github.com/huggingface/trl/pull/5067) which adds that check across 4 trainers and fixes the example notebook. That PR's description links to this one as its only context.

He asked me to split out the dtype fix as a separate PR — that became [#5073](https://github.com/huggingface/trl/pull/5073). On the reward exception handling, his take was that belongs in userland, not the trainer.

## Links

- My PR: https://github.com/huggingface/trl/pull/5064
- Maintainer's merged fix: https://github.com/huggingface/trl/pull/5067
- Follow-up dtype fix: https://github.com/huggingface/trl/pull/5073
- Issues: [#4746](https://github.com/huggingface/trl/issues/4746), [#4870](https://github.com/huggingface/trl/issues/4870), [#4451](https://github.com/huggingface/trl/issues/4451), [#5041](https://github.com/huggingface/trl/issues/5041)
