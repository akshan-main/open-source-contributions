# Diffusers: Pipeline Test Migration (QwenImage, Wan, and CogVideoX Families)

**PRs**: [#14220](https://github.com/huggingface/diffusers/pull/14220), [#14224](https://github.com/huggingface/diffusers/pull/14224), [#14228](https://github.com/huggingface/diffusers/pull/14228), [#14231](https://github.com/huggingface/diffusers/pull/14231), [#14235](https://github.com/huggingface/diffusers/pull/14235), [#14239](https://github.com/huggingface/diffusers/pull/14239), [#14240](https://github.com/huggingface/diffusers/pull/14240), [#14242](https://github.com/huggingface/diffusers/pull/14242), [#14276](https://github.com/huggingface/diffusers/pull/14276), [#14283](https://github.com/huggingface/diffusers/pull/14283), [#14284](https://github.com/huggingface/diffusers/pull/14284), [#14289](https://github.com/huggingface/diffusers/pull/14289)
**Status**: All merged

## Change

- Picked up the new pipeline-test framework the maintainer introduced in [#14113](https://github.com/huggingface/diffusers/pull/14113) (which migrated only Flux as the model case) and carried the first community batches: QwenImage, the entire Wan family, and the entire CogVideoX family — twelve test files across twelve PRs, coordinated with @sayakpaul on Slack.
- Each file moves from one `unittest.TestCase` with `params`/`batch_params` frozensets to a `<X>PipelineTesterConfig(BasePipelineTesterConfig)` holding the pipeline class, call-signature/batch/optional input-param sets, `get_dummy_components()`, and `get_dummy_inputs()`, plus `Test<X>Pipeline(Config, PipelineTesterMixin)` for fast tests and `Test<X>PipelineMemory(Config, MemoryTesterMixin)` for offload and layerwise-casting coverage.
- Existing coverage carried over rather than regenerated: expected output slices, QwenImage's VAE tiling and true-CFG-without-negative-mask tests, Wan VACE's reference-image variants, Wan Animate's replacement-inference test, and real skips with reasons (VACE raises on batched prompts; the video-to-video pipeline must run mixed precision, so full-fp16 tests stay skipped).
- Handled a Wan-specific trap in the generic optional-components test: Wan lists its required `transformer` under `_optional_components`, so the base test would null out the denoiser. The override nulls only `transformer_2` for standard Wan, and for the two-transformer Wan 2.2 variants selects which transformer is truly optional via `boundary_ratio` (`boundary_ratio=1.0` makes the 14B variant run entirely on `transformer_2`).
- Kept the suites lean: no caching tests where the old suites had none, real expected value slices at `atol=1e-3` instead of shape-only assertions, and no overrides where the base mixin already covers the behavior or the base tolerance passes.
- The CogVideoX batch is the first in the series to use the framework's cache mixins: the old text-to-video suite had PyramidAttentionBroadcast, FasterCache, and FirstBlockCache tests, and those carry over through `CacheTesterMixin` classes instead of being dropped. CogVideoX-specific tests stay too: `test_vae_tiling` (image-to-video keeps its transformer re-init for the learned positional embeddings) and `test_fused_qkv_projections`.

| PR | File | Suites |
|---|---|---|
| [#14220](https://github.com/huggingface/diffusers/pull/14220) | `test_qwenimage.py` | QwenImagePipeline |
| [#14224](https://github.com/huggingface/diffusers/pull/14224) | `test_wan.py` | WanPipeline (T2V) |
| [#14228](https://github.com/huggingface/diffusers/pull/14228) | `test_wan_image_to_video.py` | WanImageToVideoPipeline, WanFLFToVideoPipeline |
| [#14231](https://github.com/huggingface/diffusers/pull/14231) | `test_wan_vace.py` | WanVACEPipeline |
| [#14235](https://github.com/huggingface/diffusers/pull/14235) | `test_wan_video_to_video.py` | WanVideoToVideoPipeline |
| [#14239](https://github.com/huggingface/diffusers/pull/14239) | `test_wan_animate.py` | WanAnimatePipeline |
| [#14240](https://github.com/huggingface/diffusers/pull/14240) | `test_wan_22.py` | Wan 2.2 14B (two transformers) + 5B ti2v |
| [#14242](https://github.com/huggingface/diffusers/pull/14242) | `test_wan_22_image_to_video.py` | Wan 2.2 I2V 14B + 5B |
| [#14276](https://github.com/huggingface/diffusers/pull/14276) | `test_cogvideox.py` | CogVideoXPipeline (T2V, with cache mixins) |
| [#14283](https://github.com/huggingface/diffusers/pull/14283) | `test_cogvideox_image2video.py` | CogVideoXImageToVideoPipeline |
| [#14284](https://github.com/huggingface/diffusers/pull/14284) | `test_cogvideox_fun_control.py` | CogVideoXFunControlPipeline |
| [#14289](https://github.com/huggingface/diffusers/pull/14289) | `test_cogvideox_video2video.py` | CogVideoXVideoToVideoPipeline |

## What it enables

- Pipeline tests for these families now share one structure with the migrated Flux suite, so the maintainer-led refactor moved from "framework plus one example" to covering two complete video model families plus QwenImage, with the memory/offload mixins running against pipelines that never had that coverage.
- The new coverage found real bugs. Follow-up [#14269](https://github.com/huggingface/diffusers/pull/14269) (by sywangyi) fixed two pre-existing gaps these suites surfaced, not bugs in the refactors: the framework's group-offload helper didn't know about `transformer_2` and crashed on `None` optional components (Wan 2.2 was the first two-transformer pipeline through `MemoryTesterMixin`), and Wan's RoPE drifts in fp16 after save/load, fixed upstream by adding `"rope"` to `_keep_in_fp32_modules` in `transformer_wan.py`.

## Code notes

Representative before/after from `test_wan.py`:

```python
# before
class WanPipelineFastTests(PipelineTesterMixin, unittest.TestCase):
    pipeline_class = WanPipeline
    params = TEXT_TO_IMAGE_PARAMS - {"cross_attention_kwargs"}
    batch_params = TEXT_TO_IMAGE_BATCH_PARAMS
    required_optional_params = frozenset([...])
    def get_dummy_inputs(self, device, seed=0): ...

# after
class WanPipelineTesterConfig(BasePipelineTesterConfig):
    pipeline_class = WanPipeline
    required_input_params_in_call_signature = frozenset([...])
    batch_input_params = frozenset(["prompt"])
    optional_input_params = frozenset([..., "num_videos_per_prompt", ...])
    def get_dummy_components(self): ...
    def get_dummy_inputs(self):
        return {..., "generator": self.get_generator(0), "output_type": "pt"}

class TestWanPipeline(WanPipelineTesterConfig, PipelineTesterMixin):
    def test_inference(self): ...  # expected slice kept, atol=1e-3

class TestWanPipelineMemory(WanPipelineTesterConfig, MemoryTesterMixin):
    pass
```

The config no longer takes a `device` argument; generators come from `self.get_generator(seed)` and outputs are compared as torch tensors (`output_type="pt"`). Video pipelines swap the base `num_images_per_prompt` for `num_videos_per_prompt` in the optional set.

## Links

- Parent framework PR (#14113, maintainer): https://github.com/huggingface/diffusers/pull/14113
- Follow-up fix surfaced by this coverage (#14269): https://github.com/huggingface/diffusers/pull/14269
- Second model-test batch by me: [diffusers-unet-autoencoder-test-migration.md](diffusers-unet-autoencoder-test-migration.md)
