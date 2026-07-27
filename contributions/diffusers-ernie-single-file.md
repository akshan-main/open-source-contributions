# Diffusers: ERNIE-Image Single-File Loading

**PR**: [#13727](https://github.com/huggingface/diffusers/pull/13727)
**Status**: Merged (by DN6), fixes [#13722](https://github.com/huggingface/diffusers/issues/13722)

## Change

- Added `from_single_file` support to `ErnieImageTransformer2DModel`. Diffusers normally loads models from a Hub-style folder layout (config plus sharded weights); `from_single_file` is the alternate loader for one monolithic `.safetensors` checkpoint in the original key layout — the format the ComfyUI/OneTrainer/community ecosystems distribute. Calling it on the ERNIE transformer previously raised `AttributeError`.
- Three pieces, matching how Flux/Chroma wire it up: mixed `FromOriginalModelMixin` into the model class, registered the class in `SINGLE_FILE_LOADABLE_CLASSES` with a `checkpoint_mapping_fn` and `default_subfolder="transformer"`, and wrote the converter in `single_file_utils.py`. ERNIE's diffusers key names already match the original ones, so the converter is minimal by design: it only strips the `model.diffusion_model.` wrapper prefix, no remapping loop.
- No public single-file ERNIE checkpoint existed yet, so verification was a round trip: load the sharded `baidu/ERNIE-Image` transformer, save it to one safetensors file, reload via `from_single_file` — identical param count (8,033,490,048), full key-set match, sample tensor equality.

## What it enables

- Community single-file ERNIE-Image checkpoints (finetunes, redistributions in original key layout) load directly with `ErnieImageTransformer2DModel.from_single_file(...)`, the same as Flux and Chroma, instead of erroring or requiring manual key renaming and re-sharding.

## Links

- #13727: https://github.com/huggingface/diffusers/pull/13727
- Issue #13722: https://github.com/huggingface/diffusers/issues/13722
- Related: [diffusers-modular-ernie-image-pipeline.md](diffusers-modular-ernie-image-pipeline.md)
