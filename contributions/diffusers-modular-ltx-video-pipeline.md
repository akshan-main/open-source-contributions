# Diffusers: LTX Video in Modular Diffusers

**PR**: [huggingface/diffusers #13378](https://github.com/huggingface/diffusers/pull/13378)
**Status**: Merged

## Change

- Added `src/diffusers/modular_pipelines/ltx/` for LTX modular pipelines.
- Built block graphs for both text-to-video and image-to-video generation.
- Added `LTXBlocks`, `LTXImage2VideoBlocks`, and `LTXAutoBlocks` so callers can select the correct workflow through the modular pipeline system.
- Added encode, latent-prep, denoise, decode, pachifier, and postprocess blocks instead of only wrapping the existing pipeline entrypoint.
- Wired LTX into Modular Diffusers registry/import/export paths and dependency dummies.
- Added modular workflow tests under `tests/modular_pipelines/ltx/`.

## What it enables

- LTX users can now treat the video pipeline as a set of inspectable stages rather than a single opaque call.
- Researchers can replace or experiment with one stage, such as image conditioning or denoising, without copying and maintaining the whole LTX pipeline.
- `LTXAutoBlocks` gives one entry point that can choose the T2V or I2V workflow from the inputs, which is the point of adding LTX to Modular Diffusers rather than only exposing two more classes.
- Loaded components can be reused across modular workflows, so users are not forced into separate heavyweight pipeline objects for every experiment.
- Maintainers get parity-tested block boundaries around scheduler prep, image conditioning, latent packing/unpacking, and denoise-loop structure, which makes future LTX changes easier to test in isolation.

## Code notes

The new modular package breaks the pipeline into explicit stages:

- `encoders.py`: text encoding and VAE image encoding.
- `before_denoise.py`: timesteps, latent preparation, image-conditioning latents, and masks.
- `denoise.py`: loop blocks for T2V and I2V denoising.
- `decoders.py`: unpack, VAE decode, and postprocess.
- `modular_blocks_ltx.py`: workflow definitions and auto-block routing.
- `modular_pipeline.py`: LTX modular pipeline class and helper utilities.

Review feedback led to tighter workflow-map handling and generated auto docstrings, so users can discover which workflow will run and maintainers can keep LTX aligned with the conventions used by other Modular Diffusers pipelines.

## Research notes

Hugging Face's Modular Diffusers docs frame blocks as reusable pieces that can be assembled into custom pipelines, and `AutoPipelineBlocks` as the mechanism for choosing different workflows from runtime inputs. The LTX PR matters because it gives LTX users those same affordances for video generation: custom blocks, auto workflow selection, and pipeline-stage inspection.

## Links

- PR: https://github.com/huggingface/diffusers/pull/13378
- Modular Diffusers quickstart: https://huggingface.co/docs/diffusers/main/en/modular_diffusers/quickstart
- AutoPipelineBlocks docs: https://huggingface.co/docs/diffusers/main/en/modular_diffusers/auto_pipeline_blocks
- ComponentsManager docs: https://huggingface.co/docs/diffusers/main/en/modular_diffusers/components_manager
