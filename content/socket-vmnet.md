---
title: socket_vmnet
weight: 6
---

# socket_vmnet — the enforcing daemon

[`socket_vmnet`](https://github.com/tannevaled/socket_vmnet) is a fork of the
lima-vm daemon that bridges VM traffic onto macOS `vmnet`. This fork enforces a
[c-fw]({{< relref "c-fw" >}}) policy on every frame and adds a live control
plane.

```sh
socket_vmnet --acl=policy.hcl [--stateful] \
             --control-socket=/var/run/socket_vmnet.control \
             /var/run/socket_vmnet
```

- `--acl=PATH` — a `.hcl` (compiled via c-hcl) or `.json` ruleset; a bad file
  makes the daemon fail closed. `SIGHUP` hot-reloads it.
- `--stateful` — enable the connection tracker.
- `--control-socket=PATH` — the stats + control plane (below).

## Control plane

A line-delimited JSON request/response protocol on a local `AF_UNIX` socket,
served on a dedicated thread that synchronizes with the data path through the
same lock that guards the ACL swap (the kqueue/dispatch packet loop is
untouched). Each admission decision is appended to an in-memory **event ring**.

| request | reply |
|---------|-------|
| `{"cmd":"get_stats"}` | ACL counters (allow/deny frames+bytes per direction, nonip), per-rule `hits`, conntrack `capacity`/`live`/`lookups`/`hits`/`inserts` |
| `{"cmd":"get_events","since":SEQ}` | decisions with seq ≥ SEQ + `next`; each has `dir`, `verdict`, matched `rule`, `proto`, `src`/`dst`/`sport`/`dport`, `len`, `ts` |
| `{"cmd":"get_rules"}` | current ruleset `source` + `format` |
| `{"cmd":"set_acl","json":"…"}` | compile + atomically hot-swap a JSON ruleset |
| `{"cmd":"reload"}` | re-read the `--acl` file |
| `{"cmd":"reset_stats"}` | zero the counters |

```console
$ nc -U /var/run/socket_vmnet.control
{"cmd":"get_stats"}
{"ok":true,"acl":{"egress":{"allow":42,"deny":3,...},...},"rules":[{"index":0,"hits":42}],"conntrack":{...}}
```

The socket grants read **and write** access to the firewall; it is created
`0660` — protect it with filesystem permissions. The protocol handler is a pure
function (unit-tested offline) and its wire contract is checked end-to-end
against the Go client (see [fw-ui]({{< relref "fw-ui" >}})'s compat suite).
