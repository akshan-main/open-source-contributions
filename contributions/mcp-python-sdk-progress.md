# MCP Python SDK: Fix Progress Notification Routing

**PR**: [modelcontextprotocol/python-sdk #2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038)
**Status**: Merged
**Reviewer**: maxisbey

## The bug

Progress notifications from `Context.report_progress()` were silently dropped on the streamable HTTP transport. Any long-running tool (file indexing, web scraping, etc.) that called `report_progress()` — the client just never got the update. This had been open since [#953](https://github.com/modelcontextprotocol/python-sdk/issues/953), about 8 months.

It became a CI blocker when [#2002](https://github.com/modelcontextprotocol/python-sdk/pull/2002) added `related_request_id` to the progress notification protocol but didn't wire it into `report_progress()`, which broke all 20 test matrix jobs.

## The fix

The streamable HTTP transport routes progress notifications to the right SSE stream by matching `related_request_id` to the originating request. `report_progress()` wasn't passing this field, so the transport had nowhere to route it.

The fix just wires `self.request_id` through:

```python
await self._session.send_progress_notification(
    progress_token=self.meta.progress_token,
    progress=progress,
    total=total,
    message=message,
    related_request_id=self.request_id,  # added
)
```

Updated the existing test assertions that broke from the new argument, and added a unit test that mocks a `ServerRequestContext` and verifies the ID propagates through to `send_progress_notification`.

The reviewer noted that `ProgressContext.progress()` in `shared/progress.py` has the same problem — tracked in [#2021](https://github.com/modelcontextprotocol/python-sdk/issues/2021).

## Links

- PR: https://github.com/modelcontextprotocol/python-sdk/pull/2038
- Original bug: https://github.com/modelcontextprotocol/python-sdk/issues/953
- Broader tracking issue: https://github.com/modelcontextprotocol/python-sdk/issues/2021
