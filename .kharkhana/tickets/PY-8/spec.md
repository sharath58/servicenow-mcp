# PY-8 · Document the health check endpoint — spec

## Summary

Add a `#### Health Check` subsection to README.md's existing "Server-Sent Events (SSE) Mode" section, documenting how an operator confirms the SSE server is up by probing the existing `/sse` endpoint. The section must state explicitly that the server exposes **no dedicated health endpoint** (such as `/health`): the Starlette app in `src/servicenow_mcp/server_sse.py` registers only `Route("/sse", ...)` and `Mount("/messages/", ...)` (verified at base `15de01c`). No code, routes, or files other than README.md change. Stdio mode has no network surface, so the section applies to SSE mode only — its placement under the SSE section makes that implicit, and the text need not discuss stdio.

Out of scope, by name: adding a `/health` route (explicitly excluded by the ticket), Dockerfile `HEALTHCHECK` instructions or Kubernetes probe manifests (a later ticket may document those; this one only notes the curl form a probe would reuse), and any changes to `examples/` or source code.

## Changes

**README.md only.** Insert one new subsection into the existing SSE-mode documentation (currently lines 72–128), after the `#### Example` block and before `## Tool Packaging (Optional)`. Heading level `####`, matching its siblings (`#### Starting the SSE Server`, `#### Connecting to the SSE Server`, `#### Example`).

Required content of the subsection, precise enough to implement without further decisions:

1. **Heading**: `#### Health Check` (title satisfies S1's "health-check section"; exact wording of the heading may be "Health Check" or "Checking the Server Is Running" — builder's choice, but it must contain the phrase "health" so readers searching for it land here).
2. **No-dedicated-endpoint statement**: a sentence stating the server has no dedicated health endpoint such as `/health`, and that liveness is checked against the existing `/sse` endpoint. (Satisfies the second half of S1.)
3. **Concrete command**: a fenced shell block probing `/sse` on the default host/port. Because `/sse` opens a long-lived event stream, an unbounded `curl` never exits; the documented command must bound the request. Specified command:

   ```shell
   curl -s --max-time 2 -o /dev/null -w "%{http_code}\n" http://localhost:8080/sse
   ```

   with accompanying text stating that a running server prints `200` (the server accepts the connection and returns HTTP 200 with the SSE stream headers), while a down server prints `000` with a connection error. The text must note that `curl` exits after the 2-second cap because `/sse` holds the stream open by design — this is expected, not a failure.
4. **Non-default host/port note**: one sentence that operators who started the server with `--host`/`--port` substitute those values in the URL (mirrors the existing "customize the host and port" paragraph at README line 84–88).

Formatting follows the file's existing conventions: fenced code blocks, backticked endpoint paths (`/sse`, `/health`), prose in the same register as the surrounding SSE subsections. Target length: roughly 6–10 lines of prose plus one code block.

## Blast radius

- **Files**: README.md only. No source, tests, examples, Dockerfile, or packaging files change (S3).
- **Runtime**: none — a documentation-only change; no routes, behavior, or dependencies are touched.
- **Readers/operators**: operators deploying via `servicenow-mcp-sse` or the Dockerfile gain a verified liveness command; the explicit "no `/health` endpoint" statement prevents probes being pointed at a nonexistent route.
- **Risk**: the documented command could go stale if a future ticket adds a dedicated health endpoint or changes the default port; acceptable, as the section would be the natural place to update.

## Scenario coverage

| Scenario | Design element that satisfies it |
|---|---|
| S1 · README documents the liveness check | Changes items 1–3: `#### Health Check` subsection in the SSE-mode docs with a concrete `curl` command against `/sse` on the default host/port and an explicit no-dedicated-endpoint statement |
| S2 · The documented check works against a running server | Changes item 3: the `--max-time`-bounded `curl` receives HTTP 200 from a live `servicenow-mcp-sse` on `0.0.0.0:8080`; the stream-stays-open behavior is documented so the bounded exit is understood as success |
| S3 · No change outside README.md | Changes and Blast radius restrict the diff to README.md; no other file is named by this spec |

## Open questions

None. The intent pre-answers the one open question (no health endpoint exists → document that fact, do not add one), and project memory returned no decisions constraining this area.
