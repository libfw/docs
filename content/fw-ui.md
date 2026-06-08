---
title: fw-ui
weight: 7
---

# fw-ui — real-time web UI

[`libfw/fw-ui`](https://github.com/libfw/fw-ui) is the browser UI for the
firewall. It is the only process that talks to the control socket; a single
static binary serves both the API and the frontend.

- **Backend** — Go, a [Huma](https://huma.rocks) REST API (typed operations, an
  OpenAPI spec at `/openapi.json`, and a `/api/stream` server-sent-event feed).
  A poller samples the daemon counters on an interval to derive rates and tails
  the event ring. CGO-free; `internal/fwui` is held at 100% coverage.
- **Frontend** — Svelte + DaisyUI (Tailwind), built to static assets and
  embedded via `go:embed`, so `go build` needs no Node toolchain.

## API

| method | path | purpose |
|--------|------|---------|
| GET | `/api/snapshot?since=` | latest stats + rate samples + recent events |
| GET | `/api/stream` | SSE stream of snapshots |
| GET | `/api/rules` | current ruleset source + format |
| PUT | `/api/acl` | compile + hot-swap a JSON ruleset |
| POST | `/api/reload` | re-read the daemon's `--acl` file |
| POST | `/api/reset` | zero the counters |

## UI

Live packets/s and bytes/s charts (by direction and verdict), a per-rule hit
table, a filterable + pausable event log, a conntrack panel, a ruleset editor
that hot-swaps via `PUT /api/acl`, and a DaisyUI theme selector.

```sh
fw-ui --control-socket /var/run/socket_vmnet.control --listen 127.0.0.1:8849
```

A `-tags=compat` test suite drives the **real** C control plane (built as a
standalone harness) with this Go client, so the wire protocol is verified across
both implementations in CI.
