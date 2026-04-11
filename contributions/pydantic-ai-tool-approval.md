# Pydantic AI: Vercel Tool Approval and Adapter Design

**PR**: [pydantic/pydantic-ai #4283](https://github.com/pydantic/pydantic-ai/pull/4283) (closed as duplicate)
**Review**: [Comment on #3772](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902)
**Status**: Review recommendation adopted in merged [#3772](https://github.com/pydantic/pydantic-ai/pull/3772)

## Change

- Built a Vercel tool-approval implementation in #4283 covering:
  - approval-request streaming,
  - approval response parsing into approved/denied deferred tool results,
  - automatic wiring into native streaming,
  - end-to-end approval tests.
- After #4283 was closed as a duplicate, reviewed the accepted PR and pointed out a smaller adapter design.
- Suggested injecting `deferred_tool_results` through `run_stream_native()` and delegating to `super()` instead of copying the full base dispatch method.

## What it enables

- Pydantic AI users get Vercel tool approval behavior without the adapter carrying a copied version of base request-dispatch logic.
- Maintainers keep the protocol-specific customization close to the stream-native entry point where deferred tool results are injected.
- Future base adapter changes are less likely to create hidden drift in the Vercel adapter.
- The accepted PR moved back to `kwargs` plus `super()` for method overrides after the review thread.

## Review note

The suggested shape was:

```python
def run_stream_native(self, *, deferred_tool_results=None, **kwargs):
    if deferred_tool_results is None:
        deferred_tool_results = self.deferred_tool_results
    return super().run_stream_native(deferred_tool_results=deferred_tool_results, **kwargs)
```

The point was not only fewer lines. It was keeping the customization at the point where deferred tool results enter the stream, rather than duplicating a broader dispatch path that would drift as the base adapter evolves.

## Links

- My PR: https://github.com/pydantic/pydantic-ai/pull/4283
- Review comment: https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902
- Adoption reply: https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3882273449
- Merged PR: https://github.com/pydantic/pydantic-ai/pull/3772
- Feature issue: https://github.com/pydantic/pydantic-ai/issues/4279
