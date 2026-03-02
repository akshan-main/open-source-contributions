# Pydantic AI: Tool Approval + Architecture Review

**PR**: [pydantic/pydantic-ai #4283](https://github.com/pydantic/pydantic-ai/pull/4283) (closed as dup)
**Review**: [Comment on #3772](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902)
**Status**: PR closed as dup; review suggestion adopted in merged [#3772](https://github.com/pydantic/pydantic-ai/pull/3772)

## What I built

Tool approval support for the Vercel AI adapter in Pydantic AI ([#4279](https://github.com/pydantic/pydantic-ai/issues/4279)):

- Emit `ToolApprovalRequestChunk` when an agent returns `DeferredToolRequests`
- Parse `approval-responded` UI parts into `DeferredToolResults`
- Auto-wire parsed results in `run_stream_native` when not explicitly provided
- Tests for streaming, parsing, approved/denied flows

Closed as a dup of [#3772](https://github.com/pydantic/pydantic-ai/pull/3772) by bendrucker who was working on the same thing.

## The review

After my PR was closed I reviewed bendrucker's implementation. The `VercelAIAdapter` was overriding `dispatch_request` entirely — copy-pasting the full base class method just to inject `deferred_tool_results`. Any future change to `UIAdapter.dispatch_request` would need to be duplicated.

[Suggested](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) overriding `run_stream_native()` with `super()` delegation instead:

```python
def run_stream_native(self, *, deferred_tool_results=None, **kwargs):
    if deferred_tool_results is None:
        deferred_tool_results = self.deferred_tool_results
    return super().run_stream_native(deferred_tool_results=deferred_tool_results, **kwargs)
```

bendrucker refactored to use this. His [commit](https://github.com/pydantic/pydantic-ai/pull/3772/commits/de2a4bcf) is titled "Eliminate dispatch_request duplication in VercelAIAdapter" and his [reply](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3882273449): "Ok, all comments are addressed! Back to kwargs + super for the method overrides."

## Links

- My PR: https://github.com/pydantic/pydantic-ai/pull/4283
- Review comment: https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902
- Adoption reply: https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3882273449
- Merged PR: https://github.com/pydantic/pydantic-ai/pull/3772
- Issue: https://github.com/pydantic/pydantic-ai/issues/4279
