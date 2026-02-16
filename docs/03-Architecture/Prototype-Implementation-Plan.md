# Working Prototype Plan: MCP Server Connector

This plan describes a phased approach to stand up and validate a workable MCP connector for `docsmith-connect-m365`.

## Goal

Run the existing governed Microsoft 365 MCP server in either:
- **stdio mode** (local MCP clients), or
- **streamable HTTP mode** (remote MCP connector endpoint).

## Phase 0 - Baseline and prerequisites

1. Confirm tenant auth inputs and allowlist mappings.
2. Keep rollout in safe defaults:
   - `SWD_PHASE_MODE=read_only`
   - `SWD_ENABLE_WRITES=false`
   - `AUDIT_MODE=fail_closed`
3. Verify allowlist includes `MCP Audit Log` and `Governance Docs`.

## Phase 1 - Local connector baseline (stdio)

1. Configure environment and allowlist.
2. Start with `MCP_TRANSPORT=stdio` (default).
3. Connect from MCP client config and exercise:
   - `lists.query`
   - `lists.get`
   - `docs.link`
4. Validate audit rows are written for each call.

## Phase 2 - Remote connector endpoint (HTTP)

1. Switch transport:
   - `MCP_TRANSPORT=http`
   - `MCP_HTTP_HOST=0.0.0.0` (if running in container/network)
   - `MCP_HTTP_PORT=3000`
   - `MCP_HTTP_PATH=/mcp`
2. Start server and verify endpoint is live.
3. Send MCP JSON-RPC requests over streamable HTTP transport to `/mcp`.

## Phase 3 - Controlled write rollout

1. Keep `SWD_PHASE_MODE=full`, but only after read path is stable.
2. Enable writes with explicit approval:
   - `SWD_ENABLE_WRITES=true`
3. Rollout order:
   1. `Execution Inbox`
   2. `Work Orders`
   3. `Decision Log`
   4. `docs.upload`
4. Monitor duplicate `MessageId` handling for idempotency.

## Phase 4 - Hardening and production readiness

1. Front with APIM / ingress controls (IP allowlist, throttling, auth).
2. Prefer certificate auth over client secret.
3. Enforce operational alerts for:
   - audit failures
   - Graph throttling/retry spikes
   - policy denials
4. Record release checksum and deployment evidence.

## Quick run commands

```bash
# stdio mode (default)
MCP_TRANSPORT=stdio npm start

# HTTP mode
MCP_TRANSPORT=http MCP_HTTP_HOST=127.0.0.1 MCP_HTTP_PORT=3000 MCP_HTTP_PATH=/mcp npm start
```
