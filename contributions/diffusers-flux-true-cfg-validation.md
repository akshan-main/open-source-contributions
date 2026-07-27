# Diffusers: Flux True CFG and Input Validation Fixes

**PRs**: [#13957](https://github.com/huggingface/diffusers/pull/13957), [#13955](https://github.com/huggingface/diffusers/pull/13955)
**Status**: Both merged

Both PRs split findings out of the maintainer-led flux review ([#13584](https://github.com/huggingface/diffusers/issues/13584), part of the systematic review meta-issue [#13656](https://github.com/huggingface/diffusers/issues/13656)), discussed on Slack with @sayakpaul. #13957 was merged by yiyixuxu, #13955 by sayakpaul.

## Change

**#13957 — true CFG with precomputed negative embeds.** Flux models ship with distilled guidance, so "true CFG" (an actual second forward pass with a negative prompt) is opt-in via `true_cfg_scale`. In `FluxImg2ImgPipeline`, `FluxInpaintPipeline`, and `FluxControlNetPipeline` the gate was `true_cfg_scale > 1 and negative_prompt is not None` — a user passing precomputed `negative_prompt_embeds` + `negative_pooled_prompt_embeds` with no text `negative_prompt` got their embeds validated, accepted, and then silently ignored: no CFG at all, no error, just different output. The base `FluxPipeline` and `FluxKontextPipeline` already checked embeds too, so the fix brings the other three in line:

```python
# before
do_true_cfg = true_cfg_scale > 1 and negative_prompt is not None

# after
has_neg_prompt = negative_prompt is not None or (
    negative_prompt_embeds is not None and negative_pooled_prompt_embeds is not None
)
do_true_cfg = true_cfg_scale > 1 and has_neg_prompt
```

A new test asserts output actually differs between `true_cfg_scale=1.0` and `2.0` when only embeds are passed.

**#13955 — three validation gaps.**

- `FluxPipeline`/`FluxKontextPipeline` accepted `negative_prompt_embeds` with a different shape than `prompt_embeds` and blew up later mid-denoise. Now a `ValueError` with a clear message, checked only when `do_true_cfg` is active — the shapes only matter when the embeds are used.
- `FluxControlNetImg2ImgPipeline.check_inputs` had an operator-precedence bug: `height % self.vae_scale_factor * 2 != 0` parses as `(height % factor) * 2`, so invalid sizes like 72 skipped the divisibility warning and got silently resized to 64 in `prepare_latents`. Fixed to `height % (self.vae_scale_factor * 2) != 0` — this was the only flux pipeline that silently resized instead of warning.
- `FluxPriorReduxPipeline` only length-checked `prompt_embeds_scale` when `image` was a list (tensor batches went unchecked) and never validated `pooled_prompt_embeds_scale` at all. Both are now validated against the computed image batch size.

## What it enables

- True CFG works with precomputed negative embeds across all flux pipelines, matching the base pipeline — the combination that previously disabled guidance silently.
- Bad inputs fail fast where the user can act on them: mismatched embed shapes raise a named `ValueError` up front instead of a shape error deep in the model, non-divisible sizes trigger the same warning path as every other flux pipeline, and wrong-length Redux scale lists are caught for tensor batches too.

## Links

- #13957 (true CFG gate): https://github.com/huggingface/diffusers/pull/13957
- #13955 (check_inputs validation): https://github.com/huggingface/diffusers/pull/13955
- Flux review findings: https://github.com/huggingface/diffusers/issues/13584
- Systematic review meta-issue: https://github.com/huggingface/diffusers/issues/13656
