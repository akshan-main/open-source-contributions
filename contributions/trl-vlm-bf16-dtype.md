# TRL: Fix bf16/fp16 VLM Dtype Mismatch

**PR**: [huggingface/trl #5073](https://github.com/huggingface/trl/pull/5073)
**Status**: Open (split out from [#5064](https://github.com/huggingface/trl/pull/5064) at maintainer's request)

## The bug

VLM training with GRPO crashes with `RuntimeError: expected scalar type BFloat16 but found Float` when `bf16=True` or `fp16=True`. The vision encoder's layernorm expects bfloat16/float16 inputs but `pixel_values` are still float32.

Separate from the string-prompt crash fixed in [#5067](https://github.com/huggingface/trl/pull/5067). Happens even with correctly formatted prompts. Reported in [#4451](https://github.com/huggingface/trl/issues/4451).

## The fix

`_generate_and_score_completions` builds `forward_kwargs` from processor outputs (float32) and passes them to the model. The existing `_prepare_inputs` only handles DeepSpeed dtype casting, not the general case.

```python
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

The `is_floating_point` check matters because `forward_kwargs` also has integer tensors like `image_grid_thw` (grid dimensions) that would get corrupted by a float cast. Only runs when images are present, so text-only training is unaffected.

## Links

- PR: https://github.com/huggingface/trl/pull/5073
- Maintainer request: https://github.com/huggingface/trl/pull/5067#issuecomment-2853463498
- Original investigation: https://github.com/huggingface/trl/pull/5064
- Issue: https://github.com/huggingface/trl/issues/4451
