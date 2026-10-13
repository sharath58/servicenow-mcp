# PY-8 · Document the health check endpoint — plan

Documentation-only change: insert one `#### Health Check` subsection into README.md's existing "Server-Sent Events (SSE) Mode" section. No source, test, or packaging files change. There is no dedicated health endpoint in this codebase (verified: `src/servicenow_mcp/server_sse.py` registers only `/sse` and `/messages/`), and per the ticket none is added — the section documents probing the existing `/sse` endpoint.

Because S3 restricts the diff to README.md, no test files may be created or extended; S2 is proven by a manual verification step against a running server, recorded below.

## Steps

### Step 1 · Insert the Health Check subsection into README.md
Scenarios: S1, S3
Files: README.md

- **Files** — `README.md` (only). Insert a new subsection inside the "Server-Sent Events (SSE) Mode" section (currently lines 72–128), after the `#### Example` code block (ends line 128) and before `## Tool Packaging (Optional)` (line 130).
- **Satisfies** — S1, S3.
- **Tests** — none may be added (S3 limits the diff to README.md). Structural self-check the builder runs instead: `git diff --name-only 15de01c` lists exactly `README.md` plus `.kharkhana/tickets/PY-8/*`; `grep -n "#### Health Check" README.md` hits inside the SSE section; the section contains a fenced shell block with the `curl` command and the phrase stating no dedicated health endpoint (e.g. grep for `/health`).
- **Could break** — README rendering/anchors only. The insertion must not alter any existing line; sibling headings stay `####` level so the TOC hierarchy is unchanged. The scope check above protects S3.

Content of the subsection (per spec, precise enough to write without further decisions):

1. Heading `#### Health Check` — heading level matches siblings (`#### Starting the SSE Server`, `#### Connecting to the SSE Server`, `#### Example`) and contains the word "Health" so searches land on it.
2. A sentence stating the server exposes **no dedicated health endpoint** such as `/health`; liveness is checked against the existing `/sse` endpoint.
3. A fenced shell block with exactly:

   ```shell
   curl -s --max-time 2 -o /dev/null -w "%{http_code}\n" http://localhost:8080/sse
   ```

   with accompanying text: a running server prints `200`; a down server prints `000` with a connection error; `curl` exits after the 2-second cap because `/sse` holds the SSE stream open by design — expected behavior, not a failure.
4. One sentence: operators who started the server with `--host`/`--port` substitute those values in the URL (mirrors the existing customize-host-and-port paragraph at README lines 84–88).

Formatting follows the file's conventions (fenced code blocks, backticked endpoint paths, same prose register). Target length ~6–10 lines of prose plus one code block. Out of scope by name: adding a `/health` route, Dockerfile `HEALTHCHECK`, Kubernetes probe manifests, changes to `examples/` or source.

### Step 2 · Verify the documented command against a running server
Scenarios: S2, S3
Files: README.md

- **Files** — `README.md` only, and only if verification forces a correction to the command or its described output; otherwise this step changes nothing.
- **Satisfies** — S2.
- **Tests** — manual verification, since no test files are permitted (S3): start the server with `servicenow-mcp-sse` on defaults (dummy ServiceNow credentials suffice — liveness does not require a valid instance), run the README command verbatim, assert it prints `200` (an HTTP response, not a connection error); stop the server, run it again, assert it prints `000`. Then re-run `git diff --name-only 15de01c` to confirm README.md remains the only modified repository file.
- **Could break** — nothing at runtime (no code changes). If the live check contradicts the documented output, the builder corrects the README text in place rather than touching any other file; if correction would require a code change, that is drift — stop and propose it instead of widening scope.
- **Deviation (build phase)** — the live check could not be run: the lane sandbox has no Python runtime (`python3`/`pip`/`uv` absent), no `curl`, and `apt-get install` is denied by the autopilot_sandbox preset. Verified statically instead: `src/servicenow_mcp/server_sse.py:43` registers `Route("/sse", endpoint=handle_sse)`, so a GET to `/sse` on a running server returns an HTTP response (Starlette serves the route), and the documented `curl` flags are standard. The live run of the README command against `servicenow-mcp-sse` is left for a person; recorded in pr.md.

## Coverage

| Scenario | Step(s) |
|---|---|
| S1 · README documents the liveness check | Step 1 |
| S2 · The documented check works against a running server | Step 2 |
| S3 · No change outside README.md | Steps 1, 2 (scope checks in both) |
