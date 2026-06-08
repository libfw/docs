---
title: Introduction
type: docs
weight: 1
---

# libfw

**libfw** is a small family of dependency-free C libraries for building network
policy into your own programs — and the tooling around them. Everything is
zero-dependency, BSD-3-licensed, and built from source.

| component | what it is |
|-----------|------------|
| [c-hcl]({{< relref "c-hcl" >}}) | a tiny HCL-subset configuration parser in C |
| [c-fw]({{< relref "c-fw" >}}) | an embeddable stateful L3/L4 packet filter (ACL + conntrack + HCL front-end) |
| [socket_vmnet]({{< relref "socket-vmnet" >}}) | the macOS `vmnet` daemon that enforces a c-fw policy on VM traffic, with a live control plane |
| [fw-ui]({{< relref "fw-ui" >}}) | a real-time web UI (Go + Huma + Svelte/DaisyUI) for that control plane |

```
  c-hcl  ──vendored──▶  c-fw  ──vendored──▶  socket_vmnet ──UNIX socket──▶  fw-ui
 (HCL parser)        (ACL+conntrack)        (vmnet daemon)   (JSON/SSE)   (web UI)
```

Start at [Getting started]({{< relref "getting-started" >}}), or jump to a
component in the sidebar.
