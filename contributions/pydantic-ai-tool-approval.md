# Pydantic AI: Tool Approval + Architecture Review

**PR**: [pydantic/pydantic-ai #4283](https://github.com/pydantic/pydantic-ai/pull/4283) (closed as dup)
**Review**: [Comment on #3772](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902)
**Status**: PR closed as dup; review suggestion adopted in merged [#3772](https://github.com/pydantic/pydantic-ai/pull/3772)

## What I built

Tool approval support for the Vercel AI adapter ([#4279](https://github.com/pydantic/pydantic-ai/issues/4279)). The adapter needed to handle the human-in-the-loop flow: when an agent returns `DeferredToolRequests`, stream `tool-approval-request` events to the frontend, then when the frontend sends back `approval-responded` parts, parse them into `ToolApproved`/`ToolDenied` results and feed them back into the next agent run.

Built the full flow: streaming emission of approval request chunks, Pydantic models for parsing Vercel's `approval-responded` schema (both builtin and dynamic tool variants), auto-wiring of parsed results in `run_stream_native()` so users don't have to manually pass `deferred_tool_results`, and tests covering the whole thing.

Closed as a dup of [#3772](https://github.com/pydantic/pydantic-ai/pull/3772) by bendrucker who was building the same feature.

## The review

After my PR was closed I went through bendrucker's implementation. The problem: `VercelAIAdapter` was overriding `dispatch_request` in its entirety, copying the full base class method just to inject `deferred_tool_results`. That means any future change to `UIAdapter.dispatch_request` would need a parallel update in the Vercel adapter or it'd silently drift.

[Suggested](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) overriding `run_stream_native()` with `super()` delegation instead:

```python
def run_stream_native(self, *, deferred_tool_results=None, **kwargs):
    if deferred_tool_results is None:
        deferred_tool_results = self.deferred_tool_results
    return super().run_stream_native(deferred_tool_results=deferred_tool_results, **kwargs)
```

The adapter only touches the one thing it needs to and delegates everything else.

bendrucker refactored to use this. His [commit](https://github.com/pydantic/pydantic-ai/pull/3772/commits/de2a4bcf) is titled "Eliminate dispatch_request duplication in VercelAIAdapter" and his [reply](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3882273449): "Ok, all comments are addressed! Back to kwargs + super for the method overrides."

## Links

- My PR: https://github.com/pydantic/pydantic-ai/pull/4283
- Review comment: https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902
- Adoption reply: https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3882273449
- Merged PR: https://github.com/pydantic/pydantic-ai/pull/3772
- Issue: https://github.com/pydantic/pydantic-ai/issues/4279
