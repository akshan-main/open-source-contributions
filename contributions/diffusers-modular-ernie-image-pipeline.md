# Diffusers: Ernie-Image in Modular Diffusers

**PRs**: [#13498](https://github.com/huggingface/diffusers/pull/13498) (pipeline), [#13663](https://github.com/huggingface/diffusers/pull/13663) (review-finding fixes)
**Status**: Both merged

## Change

- Added `src/diffusers/modular_pipelines/ernie_image/` with `ErnieImageAutoBlocks` and `ErnieImageModularPipeline`.
- Split the pipeline into explicit stages: prompt and image encoding, timestep and latent preparation, the denoise loop, and VAE decode and postprocess.
- Wired Ernie-Image into the Modular Diffusers registry, import, and export paths, added the dependency dummies, and added modular workflow tests under `tests/modular_pipelines/ernie_image/`.
- Follow-up [#13663](https://github.com/huggingface/diffusers/pull/13663) addressed maintainer review findings from [#13577](https://github.com/huggingface/diffusers/issues/13577) (items 1, 2, and 5 per the requested scope).

## What it enables

- Ernie-Image users can treat the pipeline as inspectable stages instead of a single call, and assemble or swap individual blocks.
- Researchers can replace one stage, such as the prompt enhancer or the denoise loop, without copying and maintaining the whole pipeline.
- The follow-up fixes made the modular path behave like the standard pipeline in three places where it had drifted: prompt-enhancer skipping, VAE normalization, and latent-output handling.

## Parity

The modular blocks were checked against the standard pipeline on an A100 in bf16, 50 steps, at 1024x1024 with `baidu/ERNIE-Image`:

- Mean absolute difference: `0.000033`.
- Max absolute difference: `0.58` out of 255.

The PR links a Colab notebook that reproduces the comparison.

## Review-finding fixes (#13663)

- **Prompt-enhancer skip**: switched `ErnieImageAutoPromptEnhancerStep` to `ConditionalPipelineBlocks` so `use_pe=False` actually skips the prompt enhancer. `AutoPipelineBlocks` selected on presence rather than truthiness, so the flag was not being honored.
- **VAE normalization**: aligned the modular VAE batch-norm epsilon to the standard pipeline's hardcoded `1e-5`, which matches training. The hub config reports `1e-4`, so the modular path was normalizing differently from the reference pipeline.
- **Latent output**: restructured `output_type="latent"` to run `maybe_free_model_hooks()` and honor `return_dict`, matching the QwenImage and Flux2 pattern.

## Code notes

The package uses the same stage layout as the other Modular Diffusers pipelines:

- `encoders.py`: prompt and image encoding.
- `before_denoise.py`: timesteps and latent preparation.
- `denoise.py`: the denoise loop blocks.
- `decoders.py`: VAE decode and postprocess.
- `modular_blocks_ernie_image.py`: workflow definitions and block wiring.
- `modular_pipeline.py`: the Ernie-Image modular pipeline class.

The review fixes are the useful part of this entry: parity numbers show the blocks match, but the follow-up is where the modular path was made to match the standard pipeline on conditional steps, normalization constants, and output contracts, which is exactly where a stage-based rewrite tends to drift from the original.

## Links

- Pipeline PR: https://github.com/huggingface/diffusers/pull/13498
- Review-finding fixes: https://github.com/huggingface/diffusers/pull/13663
- Review issue: https://github.com/huggingface/diffusers/issues/13577
