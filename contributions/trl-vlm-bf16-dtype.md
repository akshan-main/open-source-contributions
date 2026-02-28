# TRL: Fix bf16/fp16 VLM Dtype Mismatch

**PR**: [huggingface/trl #5073](https://github.com/huggingface/trl/pull/5073)
**Status**: Open (requested by maintainer from [#5064](https://github.com/huggingface/trl/pull/5064) / [#5067](https://github.com/huggingface/trl/pull/5067) investigation)

## What broke

VLM training with GRPO crashes with `RuntimeError: expected scalar type BFloat16 but found Float` when `bf16=True` or `fp16=True`. The crash occurs in `torch.layer_norm` inside the vision encoder because `pixel_values` remain float32 while model weights are in the compute dtype.

Reported in [#4451](https://github.com/huggingface/trl/issues/4451) comments. This is a separate crash from the string-prompt TypeError fixed in [#5067](https://github.com/huggingface/trl/pull/5067) — it happens even with correctly formatted conversational prompts.

## Root cause

`GRPOTrainer._generate_and_score_completions` builds `forward_kwargs` from processor outputs (which are float32) and passes them to the model. The existing `_prepare_inputs` method only handles DeepSpeed-specific dtype casting — it does not cast `pixel_values` or other vision tensors to the training compute dtype. When the vision encoder's layernorm expects bfloat16/float16 inputs, the float32 tensors cause a dtype mismatch.

## Code change

Fix in `trl/trainer/grpo_trainer.py`:

```python
forward_kwargs = {k: v for k, v in prompt_inputs.items() if k not in ["input_ids", "attention_mask"]}

if self.args.bf16:
    compute_dtype = torch.bfloat16
elif self.args.fp16:
    compute_dtype = torch.float16
else:
    compute_dtype = None
if compute_dtype is not None:
    forward_kwargs = {
        k: v.to(compute_dtype) if isinstance(v, torch.Tensor) and torch.is_floating_point(v) else v
        for k, v in forward_kwargs.items()
    }
```

The `torch.is_floating_point(v)` guard ensures only floating-point tensors are cast — integer tensors like `image_grid_thw` (grid dimensions) are left unchanged.

## Provenance

This fix was identified during the [#5064](https://github.com/huggingface/trl/pull/5064) investigation. When the maintainer merged [#5067](https://github.com/huggingface/trl/pull/5067) (the string-prompt fix), I flagged on that PR that the dtype issue was still unresolved. The maintainer responded: "Yes please open a separate PR." This PR is that follow-up.

## Why it matters

bf16 is the standard training dtype for VLMs on modern GPUs. Without this fix, anyone training a vision-language model (Qwen-VL, LLaVA, etc.) with GRPO in mixed precision hits this crash. No behavioral changes for fp32 training.

## Links

- PR: https://github.com/huggingface/trl/pull/5073
- Maintainer request context: https://github.com/huggingface/trl/pull/5067#issuecomment-2853463498
- Original investigation: https://github.com/huggingface/trl/pull/5064
- Issue: https://github.com/huggingface/trl/issues/4451
