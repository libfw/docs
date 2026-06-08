---
title: c-fw
weight: 5
---

# c-fw — an embeddable L3/L4 packet filter

[`libfw/c-fw`](https://github.com/libfw/c-fw) is a dependency-free, stateful
packet filter you can embed in any C program. It matches raw ethernet frames,
so it carries no network-stack or vmnet dependency.

## Pieces

- **ACL** (`acl.{c,h}`) — a first-match-wins access-control list over
  ethernet / IPv4 / IPv6 / TCP / UDP / ICMP (source/dest MAC, CIDR, protocol,
  port or range, direction), loaded from a compact JSON, with a default action.
- **conntrack** (`conntrack.{c,h}`) — a stateful TCP/UDP tracker by normalized
  5-tuple with idle timeouts and an LRU, so return traffic of an allowed flow
  passes without an explicit reverse rule. Time is injected for determinism.
- **HCL front-end** (`acl_hcl.{c,h}`) — compiles an HCL ruleset (via
  [c-hcl]({{< relref "c-hcl" >}}), vendored as a submodule) into an ACL,
  expanding named groups over their member MACs.

## Decision + observability API

```c
// verdict + which rule matched (-1 = default / non-IP), updating counters:
bool acl_check(const struct acl *, enum acl_dir, const uint8_t *frame, size_t, int *matched);
bool acl_allows(const struct acl *, enum acl_dir, const uint8_t *frame, size_t);

// counters for a live UI:
void     acl_get_stats(const struct acl *, struct acl_stats *out); // allow/deny frames+bytes/dir, nonip
uint64_t acl_rule_hits(const struct acl *, size_t idx);            // per-rule
void     acl_reset_stats(struct acl *);
bool     acl_classify(const uint8_t *frame, size_t, struct acl_l3l4 *out); // 5-tuple for events

void conntrack_get_stats(const struct conntrack *, uint64_t now, struct conntrack_stats *out);
```

These counters and the `acl_check` rule index are what
[socket_vmnet]({{< relref "socket-vmnet" >}})'s control plane exports to the UI.

Tested under ASan + allocation fault-injection (acl ~98% line, conntrack ~99%,
100% functions).
