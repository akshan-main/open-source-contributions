# Diffusers: Model Test Migration, Second Batch (UNets, ControlNet, Autoencoders)

**PRs**: [#13832](https://github.com/huggingface/diffusers/pull/13832), [#13847](https://github.com/huggingface/diffusers/pull/13847), [#13891](https://github.com/huggingface/diffusers/pull/13891), [#13897](https://github.com/huggingface/diffusers/pull/13897), [#13898](https://github.com/huggingface/diffusers/pull/13898), [#13901](https://github.com/huggingface/diffusers/pull/13901)
**Status**: All merged

## Change

- Continued the modeling-test migration from the [first batch](diffusers-model-test-mixin-migration.md) across six more merged PRs, moving 11 model classes (14 test suites, since UNet1D carries an RL variant and UNet2D carries LDM and NCSNpp variants) onto the shared config-plus-mixin structure: four video autoencoders, the Cosmos ControlNet, and the UNet family.
- Same shape as the first batch: `<Model>TesterConfig(BaseModelTesterConfig)` with `get_init_dict()` and seeded `randn_tensor` inputs in `get_dummy_inputs()`, plus one test class per concern (`ModelTesterMixin`, `TrainingTesterMixin`, `MemoryTesterMixin`, and `AttentionTesterMixin` or the autoencoder slicing/tiling mixin where the model supports it).
- The mixin set follows model capability instead of being copied blindly: UNet1D gets only `ModelTesterMixin` (no attention processors, no gradient checkpointing), UNet2D and the Cosmos ControlNet leave out the attention mixin, and the Cosmos ControlNet keeps explicit skips with reasons for every generic helper its list-of-tensors output breaks.
- Bundled a model fix found during the Mochi migration: `AutoencoderKLMochi.forward` called `self.decode(z)` and wrapped the resulting `DecoderOutput` in a tuple when `return_dict=False`; it now forwards `return_dict` so the tuple path behaves like every other VAE, and the return annotation was corrected from `torch.Tensor | torch.Tensor` to `DecoderOutput | torch.Tensor`.
- Hub integration tests and hard-coded expected output slices stayed intact: UNet1D keeps its hopper value-function loading tests and `@slow` maestro test, UNet2D keeps the LDM/NCSNpp `from_pretrained` tests, and model-specific tests like `test_from_unet`, `test_feed_forward_chunking`, and the motion-adapter save/load tests moved over unchanged.

| PR | Suites migrated |
|---|---|
| [#13832](https://github.com/huggingface/diffusers/pull/13832) | AutoencoderKLTemporalDecoder, AutoencoderKLCosmos, AutoencoderKLKVAE, AutoencoderKLMochi (plus the Mochi `forward` fix) |
| [#13847](https://github.com/huggingface/diffusers/pull/13847) | CosmosControlNetModel |
| [#13891](https://github.com/huggingface/diffusers/pull/13891) | UNetSpatioTemporalConditionModel |
| [#13897](https://github.com/huggingface/diffusers/pull/13897) | UNet3DConditionModel, UNetMotionModel, UNetControlNetXSModel |
| [#13898](https://github.com/huggingface/diffusers/pull/13898) | UNet1DModel (standard + RL value-function) |
| [#13901](https://github.com/huggingface/diffusers/pull/13901) | UNet2DModel (standard + LDM + NCSNpp) |

## What it enables

- With this batch the whole UNet family and most video autoencoders read the same way as the rest of the migrated model tests: one config per suite, one class per concern, deterministic seeded inputs.
- Real model limitations stay visible instead of silently passing: the Cosmos ControlNet skips name the exact reason (its output is a list of control-block tensors, which the generic shape/comparison helpers can't handle), and NCSNpp keeps its layerwise-casting skips with reasons.
- The Mochi fix means `vae(sample, return_dict=False)` returns a tensor tuple instead of a tuple wrapping a `DecoderOutput`, matching the documented contract.

## Code notes

`output_shape` follows the repo convention of per-sample shape without the batch dimension: `(14, 16)` for the standard UNet1D and `(1,)` for the RL value-function. Getting the RL shape right matters more than it looks — the base `test_output` currently masks wrong shapes through an operator-precedence bug (`== ... or self.output_shape` is always truthy), so a correct shape here keeps the suite from breaking the day the base class is fixed.

The Mochi fix:

```python
# before: return_dict=False wrapped a DecoderOutput in a tuple
dec = self.decode(z)
if not return_dict:
    return (dec,)

# after: forwards return_dict, tuple path returns tensors
dec = self.decode(z, return_dict=return_dict)
```

## Links

- #13832 (temporal decoder, cosmos, kvae, mochi + fix): https://github.com/huggingface/diffusers/pull/13832
- #13847 (controlnet cosmos): https://github.com/huggingface/diffusers/pull/13847
- #13891 (unet spatiotemporal): https://github.com/huggingface/diffusers/pull/13891
- #13897 (unet 3d_condition, motion, controlnetxs): https://github.com/huggingface/diffusers/pull/13897
- #13898 (unet_1d standard + RL): https://github.com/huggingface/diffusers/pull/13898
- #13901 (unet_2d standard + LDM + NCSNpp): https://github.com/huggingface/diffusers/pull/13901
