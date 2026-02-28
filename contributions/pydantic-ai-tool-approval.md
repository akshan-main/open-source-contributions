# Pydantic AI: Vercel AI Tool Approval + Architecture Review

**PR**: [pydantic/pydantic-ai #4283](https://github.com/pydantic/pydantic-ai/pull/4283) (closed as duplicate)
**Review**: [Comment on #3772](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902)
**Status**: PR closed as dup; review suggestion adopted and shipped in merged [#3772](https://github.com/pydantic/pydantic-ai/pull/3772)

## What I built

Independently implemented tool approval support for the Vercel AI adapter in Pydantic AI (closes [#4279](https://github.com/pydantic/pydantic-ai/issues/4279)):

- Emit `ToolApprovalRequestChunk` when an agent returns `DeferredToolRequests` (one chunk per approval)
- Parse `approval-responded` UI parts into `DeferredToolResults` (schema matches Vercel AI SDK nested `approval {id, approved, reason}`)
- Auto-wire parsed approval results in `run_stream_native` when `deferred_tool_results` is not explicitly provided
- Tests for streaming emission, builtin/dynamic parsing, approved/denied extraction, and empty case

The PR was closed as a duplicate of [#3772](https://github.com/pydantic/pydantic-ai/pull/3772) by bendrucker, who was working on the same feature.

## Architecture review on #3772

After my PR was closed, I reviewed bendrucker's implementation and spotted an architectural problem: the `VercelAIAdapter` was overriding `dispatch_request` in its entirety — duplicating the full base class method just to inject `deferred_tool_results`. That means every future change to `UIAdapter.dispatch_request` would need a parallel update in the Vercel adapter, or it'd silently fall out of sync.

I [suggested](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902) overriding `run_stream_native()` instead and using `**kwargs` + `super()` delegation:

```python
def run_stream_native(self, *, deferred_tool_results=None, **kwargs):
    if deferred_tool_results is None:
        deferred_tool_results = self.deferred_tool_results
    return super().run_stream_native(deferred_tool_results=deferred_tool_results, **kwargs)
```

This eliminates the duplication entirely — the adapter only touches the one thing it needs to (wiring deferred results) and delegates everything else to the base class.

bendrucker refactored the PR to use exactly this pattern. His [commit](https://github.com/pydantic/pydantic-ai/pull/3772/commits/de2a4bcf) is titled "Eliminate dispatch_request duplication in VercelAIAdapter" and his [reply](https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3882273449) in the thread: "Ok, all comments are addressed! Back to kwargs + super for the method overrides." The merged code includes the `run_stream_native` override and `**kwargs` forwarding throughout.

## Why it matters

I can build production features at the level of pydantic-ai code independently, and I review other people's implementations for architectural quality beyond just correctness. The `super()` delegation pattern prevents the Vercel adapter from silently drifting out of sync with the base class on every future change.

## Links

- My PR: https://github.com/pydantic/pydantic-ai/pull/4283
- Review comment: https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3880128902
- Adoption reply: https://github.com/pydantic/pydantic-ai/pull/3772#issuecomment-3882273449
- Adoption commit: https://github.com/pydantic/pydantic-ai/pull/3772/commits/de2a4bcf
- Merged PR: https://github.com/pydantic/pydantic-ai/pull/3772
- Issue: https://github.com/pydantic/pydantic-ai/issues/4279
