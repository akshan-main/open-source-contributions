# TRL: GRPO Multimodal Prompt Crash Analysis

**PR**: [huggingface/trl #5064](https://github.com/huggingface/trl/pull/5064)
**Status**: Open; prompt guard landed in [#5067](https://github.com/huggingface/trl/pull/5067)

## Change

- Traced VLM GRPO failures inside `GRPOTrainer._generate_and_score_completions()` and `_calculate_rewards()`.
- Identified that pre-templated string prompts were being passed into `prepare_multimodal_messages()`, which expects role/content message dicts.
- Identified a separate mixed-precision path where image tensors stayed `float32` while the model ran in bf16/fp16.
- Identified reward callback exception behavior as a separate training reliability question.

## What it enables

- TRL maintainers could separate a vague “VLM GRPO crashes” problem into specific fixes: prompt-format misuse, image tensor dtype, and reward callback policy.
- Users got a clearer failure mode for pre-templated string prompts instead of a confusing low-level `TypeError`.
- The prompt-format analysis led into the merged guard in [#5067](https://github.com/huggingface/trl/pull/5067).
- Review separated the broad patch into smaller decisions, which avoided quietly changing reward-function semantics while still preserving the valid dtype fix path.
- The dtype part became the focused follow-up PR [#5073](https://github.com/huggingface/trl/pull/5073).

## Code notes

When images are present, the trainer prepares multimodal messages before tokenization. Pre-templated strings break that path because message preparation expects list/dict structure:

```python
prepare_multimodal_messages(prompt, image_list)
```

The proposed guard was:

```python
prepare_multimodal_messages(prompt, image_list) if isinstance(prompt, list) else prompt
```

The same investigation found that image tensors in `forward_kwargs` needed compute-dtype handling, but only for floating tensors so metadata tensors are not corrupted.

## Links

- My PR: https://github.com/huggingface/trl/pull/5064
- Maintainer prompt fix: https://github.com/huggingface/trl/pull/5067
- Focused dtype PR: https://github.com/huggingface/trl/pull/5073
- Related issues: https://github.com/huggingface/trl/issues/4746, https://github.com/huggingface/trl/issues/4870, https://github.com/huggingface/trl/issues/4451, https://github.com/huggingface/trl/issues/5041
