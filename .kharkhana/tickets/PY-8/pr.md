# docs: document the SSE health check in README

## What changed

Added a `#### Health Check` subsection to README.md inside the "Server-Sent Events (SSE) Mode" section, after the `#### Example` block. It states that the server exposes **no dedicated health endpoint** such as `/health`, and documents a liveness probe against the existing `/sse` endpoint:

```shell
curl -s --max-time 2 -o /dev/null -w "%{http_code}\n" http://localhost:8080/sse
```

The section explains the expected output (`200` when up, `000` on connection error when down), that `curl` exits at the 2-second cap because `/sse` holds the SSE stream open by design, and that operators substitute their `--host`/`--port` values in the URL. No health endpoint was added, per the ticket.

## Steps done

### Step 1 · Insert the Health Check subsection (S1, S3) — commit `40ef06c`

No test files permitted (S3 limits the diff to README.md); structural self-checks ran instead:

```
$ git diff --name-only 15de01c
.kharkhana/tickets/PY-8/intent.md
.kharkhana/tickets/PY-8/plan.md
.kharkhana/tickets/PY-8/spec.md
README.md

$ grep -n "#### Health Check" README.md
130:#### Health Check

$ grep -n "/health" README.md
132:The server exposes no dedicated health endpoint such as `/health`; liveness is checked against the existing `/sse` endpoint:
```

The heading sits inside the SSE section (lines 72–140), sibling headings remain `####`, and no existing line was altered (12 insertions, 0 deletions).

### Step 2 · Verify the command against a running server (S2, S3) — commit `4c102da`

**Deviation — live check not run.** The lane sandbox has no Python runtime (`python3`, `pip`, `uv` all absent), no `curl`, no `docker`, and `apt-get install` is denied by the autopilot_sandbox preset, so `servicenow-mcp-sse` could not be started. Static verification instead:

```
$ grep -n -E "Route|/sse|/messages" src/servicenow_mcp/server_sse.py
18:from starlette.routing import Mount, Route
26:    sse = SseServerTransport("/messages/")
43:            Route("/sse", endpoint=handle_sse),
44:            Mount("/messages/", app=sse.handle_post_message),
```

`/sse` is a registered Starlette route, so a GET receives an HTTP response from a running server; the `curl` flags used are standard. plan.md was updated in the same commit to record the deviation.

## Scenarios covered

| Scenario | Status |
|---|---|
| S1 · README documents the liveness check | Done (Step 1) |
| S2 · The documented check works against a running server | Statically verified only — live run left for a person (see below) |
| S3 · No change outside README.md | Done — scope check in both steps; README.md is the only modified repository file besides the ticket artifacts |

## Left for a person

- Run the S2 live check in an environment with Python: start `servicenow-mcp-sse` (dummy ServiceNow credentials suffice), run the README command verbatim, confirm it prints `200`; stop the server and confirm `000`.
- A `reference` memory fact was proposed (prop_01M4VHFXYN685PQP2C01ZFF8K2) recording that this lane lacks a Python runtime and installs are policy-blocked.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
