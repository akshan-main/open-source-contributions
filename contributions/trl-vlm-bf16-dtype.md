# TRL: bf16/fp16 VLM Image Tensor Dtype Fix

**PR**: [huggingface/trl #5073](https://github.com/huggingface/trl/pull/5073)
**Status**: Open; split from [#5064](https://github.com/huggingface/trl/pull/5064)

## Change

- Added compute-dtype handling for image-related tensors in the GRPO VLM path.
- Derived target dtype from training flags: `bf16` maps to `torch.bfloat16`, `fp16` maps to `torch.float16`.
- Cast only floating-point tensors in `forward_kwargs`.
- Left integer metadata tensors untouched.

## What it enables

- Users training VLMs with bf16/fp16 can avoid vision-encoder dtype crashes caused by `float32` image tensors.
- The fix does not corrupt metadata tensors such as `image_grid_thw`, because only floating tensors are cast.
- The patch is scoped to image-present VLM training and does not affect text-only GRPO paths.

## Code notes

The important guard is `torch.is_floating_point(v)`:

```python
if compute_dtype is not None:
    forward_kwargs = {
        k: v.to(compute_dtype) if isinstance(v, torch.Tensor) and torch.is_floating_point(v) else v
        for k, v in forward_kwargs.items()
    }
```

That makes the fix narrow: image pixels can follow the model compute dtype, while shape/grid metadata remains valid.

## Links

- PR: https://github.com/huggingface/trl/pull/5073
- Original investigation: https://github.com/huggingface/trl/pull/5064
- Related issue: https://github.com/huggingface/trl/issues/4451
