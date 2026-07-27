# Diffusers: Bria FIBO Crash Fixes

**PR**: [#13981](https://github.com/huggingface/diffusers/pull/13981)
**Status**: Merged (by yiyixuxu)

Four runtime bugs from the maintainer-led bria_fibo review ([#13618](https://github.com/huggingface/diffusers/issues/13618)), discussed on Slack with @yiyixuxu. Net −122 lines: two of the fixes are deletions of code paths that never worked.

## Change

- **`guidance_embeds=True` couldn't construct the model.** `BriaFiboTransformer2DModel` built its guidance embedder without the required `time_theta` argument, so any config or checkpoint with `guidance_embeds=True` hit a `TypeError` at construction. Also changed `if guidance:` (ambiguous tensor truthiness) to `if guidance is not None:`.
- **`prompt_embeds`/`negative_prompt_embeds` were public but unusable.** `encode_prompt()` only produced the per-layer text-encoder embeddings the FIBO transformer needs when encoding `prompt` itself, so passing precomputed `prompt_embeds` raised `UnboundLocalError`, and `negative_prompt_embeds` was recomputed from `negative_prompt` and silently ignored. Both arguments were removed from `__call__`, `encode_prompt`, and `check_inputs` of both pipelines rather than half-fixed — `prompt` is now required.
- **Tensor `image` inputs crashed `BriaFiboEditPipeline`.** The preprocessing gate read `self.latent_channels`, an attribute that doesn't exist, so any `image=torch.Tensor(...)` raised `AttributeError`. The latent-channel special case is gone; images are always resized and preprocessed.
- **`num_images_per_prompt > 1` returned malformed output.** The decode loop postprocessed each latent separately and stacked the `(1, H, W, C)` results into `(N, 1, H, W, C)` arrays (nested lists for PIL). Now the batched latent tensor is decoded once and postprocessed once, returning `(N, H, W, C)` arrays and flat PIL lists.

## What it enables

- FIBO checkpoints with guidance embeddings can be constructed and run at all.
- Tensor image inputs and multi-image generation work in both `BriaFiboPipeline` and `BriaFiboEditPipeline`.
- The API stops advertising embed arguments that could only crash or silently do nothing — a smaller honest surface instead of a larger broken one.

## Code notes

The multi-image fix, from loop-per-latent to one batched decode:

```python
# before: per-latent decode, stacked into (N, 1, H, W, C)
images = [self.image_processor.postprocess(self.vae.decode(l / std + mean, ...), ...) for l in latents]

# after: one decode, one postprocess
latents_scaled = torch.cat([latent / latents_std + latents_mean for latent in latents], dim=0)
image = self.vae.decode(latents_scaled, return_dict=False)[0]
image = self.image_processor.postprocess(image.squeeze(dim=2), output_type=output_type)
```

## Links

- #13981: https://github.com/huggingface/diffusers/pull/13981
- bria_fibo review findings: https://github.com/huggingface/diffusers/issues/13618
