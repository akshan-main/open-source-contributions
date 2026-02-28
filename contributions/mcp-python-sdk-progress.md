# MCP Python SDK: Fix Progress Notification Routing

**PR**: [modelcontextprotocol/python-sdk #2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038)
**Status**: Merged
**Reviewer**: maxisbey (approved)

## What broke

Progress notifications sent via `Context.report_progress()` were silently dropped on the streamable HTTP transport — the transport most production MCP deployments use. Long-running tools (file indexing, web scraping, model inference) would call `report_progress()` and the client would never receive the update. The bug had been open since [#953](https://github.com/modelcontextprotocol/python-sdk/issues/953).

It surfaced as a CI-blocking regression when [#2002](https://github.com/modelcontextprotocol/python-sdk/pull/2002) added `related_request_id` to the progress notification protocol but didn't wire it into `report_progress()`, failing all 20 test matrix jobs.

## Root cause

The streamable HTTP transport routes progress notifications to the correct SSE stream by matching `related_request_id` to the originating request. `Context.report_progress()` called `send_progress_notification()` without this field, so the transport had no way to associate the notification — it was silently dropped.

## What I did

Traced the notification flow from `Context.report_progress()` through `send_progress_notification()` to understand how the streamable HTTP transport routes SSE events. The fix wires `self.request_id` into the notification so the transport can route it correctly:

```python
await self._session.send_progress_notification(
    progress_token=self.meta.progress_token,
    progress=progress,
    total=total,
    message=message,
    related_request_id=self.request_id,  # added
)
```

Updated the existing test assertions in `test_176_progress_token.py` that were now failing because they didn't expect the new argument. Then wrote a dedicated unit test that sets up a mocked `ServerRequestContext` with a known `request_id`, calls `report_progress()`, and verifies the ID propagates correctly through to `send_progress_notification`. All 27 CI checks passed — 20 test matrix jobs across Python 3.10–3.14, Ubuntu + Windows, locked + lowest-direct dependencies, plus conformance tests and pre-commit.

The reviewer confirmed this was a known bug open for ~8 months since #953 and noted that `ProgressContext.progress()` in `shared/progress.py` has the same issue — the broader architectural problem of every `Context` method having to manually thread `related_request_id` is now tracked in [#2021](https://github.com/modelcontextprotocol/python-sdk/issues/2021).

## Links

- PR: https://github.com/modelcontextprotocol/python-sdk/pull/2038
- Original bug: https://github.com/modelcontextprotocol/python-sdk/issues/953
- Broader tracking issue: https://github.com/modelcontextprotocol/python-sdk/issues/2021
