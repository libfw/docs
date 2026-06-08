---
title: Getting started
weight: 2
---

# Getting started

## Build the firewall daemon

```sh
git clone --recursive https://github.com/tannevaled/socket_vmnet
cd socket_vmnet
make                       # links c-fw + c-hcl from third_party/
```

`--recursive` matters: `socket_vmnet` vendors `c-fw`, which vendors `c-hcl`.

## Run it with a policy and a control socket

```sh
sudo socket_vmnet \
  --acl=/etc/socket_vmnet/acl.hcl \
  --control-socket=/var/run/socket_vmnet.control \
  /var/run/socket_vmnet
```

The ACL is authored in HCL (see [c-fw]({{< relref "c-fw" >}})); `--stateful`
adds connection tracking so return traffic of an allowed flow passes.

## Run the web UI

```sh
git clone https://github.com/libfw/fw-ui
cd fw-ui
(cd web && npm install && npm run build)     # Svelte/DaisyUI -> embedded assets
CGO_ENABLED=0 go build ./cmd/fw-ui
./fw-ui --control-socket /var/run/socket_vmnet.control --listen 127.0.0.1:8849
# open http://127.0.0.1:8849
```
