# Diffusers: HunyuanVideo 1.5 I2V Pixel-Resolution Fix

**PR**: [huggingface/diffusers #13440](https://github.com/huggingface/diffusers/pull/13440)
**Status**: Merged

## Change

- Fixed dimension shadowing in `prepare_cond_latents_and_mask()`.
- Preserved the caller-provided pixel-space `height` and `width` for image preprocessing.
- Renamed latent tensor dimensions to `latent_height` and `latent_width`.
- Used latent dimensions only for latent tensor and mask allocation.

## What it enables

- HunyuanVideo 1.5 I2V users get conditioning based on the pixel resolution they asked for, not on the smaller latent tensor dimensions.
- The image-conditioning path no longer silently preprocesses the input image at latent size before VAE encoding.
- Maintainers get a clearer boundary between public pipeline inputs and internal latent tensor sizes, which reduces the chance of similar bugs in video conditioning code.

## Code notes

The problematic line overwrote pixel-space dimensions:

```python
batch, channels, frames, height, width = latents.shape
```

The fix keeps latent dimensions explicit:

```python
batch, channels, frames, latent_height, latent_width = latents.shape
```

Then `height` and `width` continue to drive `_get_image_latents(...)`, while `latent_height` and `latent_width` are used for `latent_mask`.

## Links

- PR: https://github.com/huggingface/diffusers/pull/13440
