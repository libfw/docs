---
title: Architecture
weight: 3
---

# Architecture

A frame entering or leaving a guest is the natural chokepoint, so policy is
enforced in the daemon — not in the host firewall.

```
 guest ⇄ socket_vmnet (vmnet daemon)
              │  every frame
              ▼
        frame_allowed()
              │ acl_check() ──▶ c-fw ACL (first-match) ──▶ c-hcl (HCL ruleset)
              │ conntrack    ──▶ c-fw conntrack (stateful return traffic)
              ▼
        allow / drop  + record event
              │
              ▼  --control-socket (dedicated thread, JSON over AF_UNIX)
        control plane ──────────────▶ fw-ui (Go: Huma REST + SSE) ──▶ browser
```

- **c-hcl** parses the declarative HCL ruleset into an AST.
- **c-fw** compiles it to a first-match ACL and matches each frame
  (MAC / CIDR / proto / port / direction), with an optional connection tracker;
  it also keeps per-rule and aggregate counters and a recent-decision event ring.
- **socket_vmnet** is the only privileged piece: it owns the `vmnet` interface
  and calls `frame_allowed()` on every frame. Its control socket exposes the
  counters/events and accepts live rule edits.
- **fw-ui** connects to that socket, derives rates, streams to the browser over
  SSE, and proxies rule edits — it never needs privileges itself.

The libraries carry no vmnet dependency, so c-fw/c-hcl embed in any program.
