# Diffusers: HunyuanVideo 1.5 in Modular Diffusers

**PR**: [huggingface/diffusers #13389](https://github.com/huggingface/diffusers/pull/13389)
**Status**: Merged

## Change

- Added `src/diffusers/modular_pipelines/hunyuan_video1_5/` for HunyuanVideo 1.5 modular pipelines.
- Built block graphs for both text-to-video (`HunyuanVideo15Blocks`) and image-to-video (`HunyuanVideo15Image2VideoBlocks`), plus `HunyuanVideo15ModularPipeline`.
- Split the pipeline into explicit stages: text and image encoding, timestep and latent preparation, the denoise loop, and VAE decode and postprocess.
- Wired HunyuanVideo 1.5 into the Modular Diffusers registry, import, and export paths, and added the dependency dummies.
- Added modular workflow tests under `tests/modular_pipelines/hunyuan_video1_5/`.
- Addresses [#13295](https://github.com/huggingface/diffusers/issues/13295), the HunyuanVideo 1.5 contribution.

## What it enables

- HunyuanVideo 1.5 users can treat the video pipeline as inspectable stages instead of a single call, and run either T2V or I2V through the modular system.
- Researchers can replace or experiment with one stage, such as image conditioning or the denoise loop, without copying and maintaining the whole pipeline.
- Loaded components can be reused across modular workflows, so users are not forced into separate heavyweight pipeline objects for every experiment.
- Maintainers get tested block boundaries around encoding, latent preparation, denoising, and decoding for a large video model, which makes future changes easier to test in isolation.

## Parity

The modular blocks were checked against the standard pipelines on a Colab GPU at the same seed and settings:

- T2V: mean absolute difference `0.000000` against `HunyuanVideo15Pipeline`.
- I2V: mean absolute difference `0.000000` against `HunyuanVideo15ImageToVideoPipeline`.

The PR includes the full reproduction code for both paths, so the parity claim can be rerun rather than taken on trust.

## Code notes

The new modular package follows the same stage layout used by other Modular Diffusers pipelines:

- `encoders.py`: text encoding and image encoding.
- `before_denoise.py`: timesteps, latent preparation, and image-conditioning inputs.
- `denoise.py`: loop blocks for the T2V and I2V denoising paths.
- `decoders.py`: VAE decode and postprocess.
- `modular_blocks_hunyuan_video1_5.py`: workflow definitions and block wiring.
- `modular_pipeline.py`: the HunyuanVideo 1.5 modular pipeline class.

## Links

- PR: https://github.com/huggingface/diffusers/pull/13389
- Tracking issue: https://github.com/huggingface/diffusers/issues/13295
