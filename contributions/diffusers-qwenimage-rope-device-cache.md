# Diffusers: QwenImage-Family RoPE Device Cache

**PR**: [huggingface/diffusers #13406](https://github.com/huggingface/diffusers/pull/13406)
**Status**: Merged

## Change

- Profiled the QwenImage eager inference path and traced repeated CPU-to-GPU transfers inside RoPE frequency lookup.
- Added `_get_device_freqs(device)` in both `QwenEmbedRope` and `QwenEmbedLayer3DRope`.
- Used `@lru_cache_unless_export(maxsize=None)` so device copies are reused without breaking export paths.
- Replaced repeated per-step `.to(device)` calls in:
  - `forward()`
  - `_compute_video_freqs()`
  - `_compute_condition_freqs()`

## What it enables

- Users get faster eager-mode Qwen generation without changing model outputs or forcing `torch.compile`.
- The measured eager path removes about `76ms` of `cudaStreamSynchronize` per `transformer_forward` call.
- At 20 inference steps, that is roughly `~1.5s` less CPU-GPU sync overhead per run.
- The fix is in the shared QwenImage transformer path, so one patch improves `QwenImage`, `QwenImageEdit`, `QwenImageEditPlus`, and `QwenImageLayered`.
- Follow-up profiling on QwenImageEdit showed the remaining syncs were separate from RoPE: masked boolean indexing and VAE image mean transfer. That turned the performance work into a bounded finding instead of guessing at the next bottleneck.

## Code notes

The central helper is intentionally small:

```python
@lru_cache_unless_export(maxsize=None)
def _get_device_freqs(self, device: torch.device) -> tuple[torch.Tensor, torch.Tensor]:
    return self.pos_freqs.to(device), self.neg_freqs.to(device)
```

The first call materializes device copies; later denoising steps reuse them. The computation is unchanged, only the repeated transfer and synchronization are removed.

## Links

- PR: https://github.com/huggingface/diffusers/pull/13406
- Profiling note: https://github.com/huggingface/diffusers/pull/13406#issuecomment-4185515641
- QwenImageEdit follow-up profiling: https://github.com/huggingface/diffusers/pull/13406#issuecomment-4224964911
