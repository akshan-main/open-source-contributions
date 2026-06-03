# Diffusers: Model Test Architecture Refactoring

**PRs**: [#13849](https://github.com/huggingface/diffusers/pull/13849), [#13845](https://github.com/huggingface/diffusers/pull/13845), [#13840](https://github.com/huggingface/diffusers/pull/13840), [#13835](https://github.com/huggingface/diffusers/pull/13835), [#13834](https://github.com/huggingface/diffusers/pull/13834), [#13826](https://github.com/huggingface/diffusers/pull/13826)
**Status**: All merged

## Change

- Took on a chunk of the diffusers maintainer-led modeling-test migration (following [#13369](https://github.com/huggingface/diffusers/pull/13369) and [#13153](https://github.com/huggingface/diffusers/pull/13153)) and carried it across six merged PRs, moving 12 model test suites onto the shared config-plus-mixin structure: 11 autoencoders and the Sana transformer.
- Replaced each old single-class `unittest` suite with a `<Model>TesterConfig` (subclass of `BaseModelTesterConfig`) that holds the model class, inputs, and shapes, plus separate test classes per concern that pull in `ModelTesterMixin`, `TrainingTesterMixin`, `MemoryTesterMixin`, and the autoencoder `NewAutoencoderTesterMixin`. The Sana transformer suite uses `AttentionTesterMixin` in place of the autoencoder mixin.
- Swapped `prepare_init_args_and_inputs_for_common` and the `dummy_input` property for `get_init_dict()` and `get_dummy_inputs()`, and moved random inputs from `floats_tensor` to a seeded `randn_tensor` so each suite builds the same inputs on every run.
- Converted `unittest` assertions and skips to pytest (`assert`, `@pytest.mark.skip`), and kept the `@slow` integration classes and model-specific skips intact with their reasons.

| PR | Suites migrated |
|---|---|
| [#13849](https://github.com/huggingface/diffusers/pull/13849) | VQModel, AutoencoderKLKVAEVideo, AutoencoderOobleck, ConsistencyDecoderVAE, AutoencoderTiny, AutoencoderVidTok |
| [#13845](https://github.com/huggingface/diffusers/pull/13845) | AsymmetricAutoencoderKL, AutoencoderKLLTXVideo |
| [#13840](https://github.com/huggingface/diffusers/pull/13840) | AutoencoderKLCogVideoX |
| [#13835](https://github.com/huggingface/diffusers/pull/13835) | AutoencoderKLHunyuanVideo |
| [#13834](https://github.com/huggingface/diffusers/pull/13834) | AutoencoderKLMagvit |
| [#13826](https://github.com/huggingface/diffusers/pull/13826) | SanaTransformer2DModel |

## What it enables

- All of these suites now follow the same config-plus-mixin shape as the rest of the migrated model tests, so a contributor reads one structure across files instead of a different convention per suite.
- Each concern lives in its own class (model behavior, training, memory, and slicing/tiling for autoencoders, or attention for the Sana transformer), so a failure points at the concern that broke instead of one large mixed class.
- Seeded `randn_tensor` inputs make the suites deterministic by construction rather than relying on global seeding side effects.
- Model-specific behavior is preserved rather than dropped: the KVAE video suite keeps its non-deterministic backward workaround and its gradient-checkpointing cache skips, and the Sana suite leaves out the compile mixin because `SanaTransformer2DModel` does not set `_repeated_blocks`.

## Code notes

Each migration splits one class into a shared config and one test class per concern. Before:

```python
class VQModelTests(ModelTesterMixin, AutoencoderTesterMixin, unittest.TestCase):
    model_class = VQModel
    main_input_name = "sample"

    def prepare_init_args_and_inputs_for_common(self):
        init_dict = {...}
        inputs_dict = self.dummy_input
        return init_dict, inputs_dict
```

After:

```python
class VQModelTesterConfig(BaseModelTesterConfig):
    @property
    def model_class(self):
        return VQModel

    def get_init_dict(self) -> dict:
        return {...}

    def get_dummy_inputs(self) -> dict:
        image = randn_tensor((batch_size, num_channels, *sizes), generator=self.generator, device=torch_device)
        return {"sample": image}


class TestVQModel(VQModelTesterConfig, ModelTesterMixin): ...
class TestVQModelTraining(VQModelTesterConfig, TrainingTesterMixin): ...
class TestVQModelMemory(VQModelTesterConfig, MemoryTesterMixin): ...
class TestVQModelSlicingTiling(VQModelTesterConfig, NewAutoencoderTesterMixin): ...
```

The config carries everything a suite needs to build the model and its inputs, and each test class pulls only the tests for its concern. Skips that reflect a real model limitation stayed as skips with their reasons, so the work changed structure without dropping or silently passing tests.

## Links

- #13849 (vq, kvae_video, oobleck, consistency_decoder, tiny, vidtok): https://github.com/huggingface/diffusers/pull/13849
- #13845 (asymmetric_kl, ltx_video): https://github.com/huggingface/diffusers/pull/13845
- #13840 (cogvideox): https://github.com/huggingface/diffusers/pull/13840
- #13835 (hunyuan_video): https://github.com/huggingface/diffusers/pull/13835
- #13834 (magvit): https://github.com/huggingface/diffusers/pull/13834
- #13826 (sana transformer): https://github.com/huggingface/diffusers/pull/13826
