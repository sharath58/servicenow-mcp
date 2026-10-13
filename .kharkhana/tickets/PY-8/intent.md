# PY-8 · Document the health check endpoint

## Problem

Operators running the ServiceNow MCP server in SSE mode (`servicenow-mcp-sse`, listening on `0.0.0.0:8080` by default) have no documented way to verify the server is up. The README covers starting the server and connecting a client, but not a liveness check.

**There is no dedicated health endpoint in this codebase.** The Starlette app in `src/servicenow_mcp/server_sse.py` registers only two routes: `Route("/sse", ...)` and `Mount("/messages/", ...)`. Per the ticket, we will not add one; the documentation must work with the endpoints that exist. (Stdio mode has no network surface, so a health check only applies to SSE mode.)

## Proposed outcome

- README.md gains a short "Health check" (or similarly titled) subsection under the existing SSE-mode documentation, describing how to confirm the server is up by probing the existing `/sse` endpoint (e.g. a `curl` request that receives an HTTP response from the running server).
- The section explicitly notes that the server has no dedicated health endpoint such as `/health`, so readers are not left hunting for one.
- No source code, routes, or files other than README.md change.

## Affected users and systems

- Operators and integrators deploying the SSE server (including via the Dockerfile) who need a liveness probe or a quick manual check.
- Only `README.md` changes. No packages, services, or data are affected.

## Constraints

- Ticket scope: keep the change to README.md; do not add a health endpoint (ticket PY-8 description).
- The documented check must use only the routes that exist today (`/sse`, `/messages/`) — verified against `src/servicenow_mcp/server_sse.py` at base `15de01c`.
- Project memory search returned no inherited decisions or constraints for this area.

## Open questions

- None. The ticket pre-answers the key question (no health endpoint exists → document that fact, do not add one).

## Validation scenarios

### S1 · README documents the liveness check
- **Given** the repository at this ticket's branch head
- **When** a reader opens README.md and searches for a health-check section in the SSE-mode documentation
- **Then** a section exists that shows a concrete command (e.g. `curl`) against the `/sse` endpoint on the configured host and port, and states that no dedicated health endpoint exists

### S2 · The documented check works against a running server
- **Given** the SSE server started with `servicenow-mcp-sse` on its default host and port
- **When** the command from the README's health-check section is run
- **Then** it receives an HTTP response from the server (confirming liveness) rather than a connection error

### S3 · No change outside README.md
- **Given** the diff of this ticket's build phase against base `15de01c`
- **When** the changed files are listed
- **Then** README.md is the only modified repository file
