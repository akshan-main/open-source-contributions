# MCP Python SDK: Progress Notification Routing

**PR**: [modelcontextprotocol/python-sdk #2038](https://github.com/modelcontextprotocol/python-sdk/pull/2038)
**Status**: Merged

## Change

- Passed `related_request_id=self.request_id` from `Context.report_progress()` into `send_progress_notification()`.
- Updated existing progress-token assertions for the new argument.
- Added regression coverage proving `Context.report_progress()` carries the originating request ID.

## What it enables

- MCP clients using streamable HTTP can show progress for long-running tools on the correct request stream.
- Tool authors can call `report_progress()` and expect the transport to preserve the relationship between the progress event and the request that started the work.
- Without `related_request_id`, clients can miss progress entirely even though the server emitted it.
- The maintainer confirmed the code fix and tied it to the broader request-ID threading issue in the SDK.

## Code notes

The important call path is small but protocol-critical:

```python
await self._session.send_progress_notification(
    progress_token=self.meta.progress_token,
    progress=progress,
    total=total,
    message=message,
    related_request_id=self.request_id,
)
```

The regression test constructs a `ServerRequestContext` with `request_id="req-abc-123"` and asserts that `report_progress()` forwards it.

## Links

- PR: https://github.com/modelcontextprotocol/python-sdk/pull/2038
- Original issue: https://github.com/modelcontextprotocol/python-sdk/issues/953
- Related tracking: https://github.com/modelcontextprotocol/python-sdk/issues/2021
