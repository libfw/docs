---
title: c-hcl
weight: 4
---

# c-hcl — an HCL-subset parser in C

[`libhcl/c-hcl`](https://github.com/libhcl/c-hcl) parses the **declarative
subset** of HCL native syntax — bodies of attributes and labeled blocks, with
string / number / bool / null / list values — into an immutable AST with
accessors. It is one `hcl.c` + `ast.c`, zero third-party dependencies.

```hcl
default_action = "deny"

rule {
  action    = "allow"
  direction = "egress"
  proto     = "tcp"
  dst_port  = 443
}
```

```c
#include "hcl.h"
hcl_doc *doc = hcl_parse(src, len, err, sizeof err);
const hcl_body *root = hcl_doc_root(doc);
const hcl_value *v = hcl_body_attr(root, "default_action");
const hcl_block *r = hcl_body_block_at(root, "rule", 0);
hcl_free(doc);
```

It is **not** HCL2: no expressions, interpolation, functions or heredocs — it is
configuration, not computation. For the full HCL2 language (a cty value model,
the Terraform function set, conversions and diagnostics) see the sibling
[`libhcl/c-hcl2`](https://github.com/libhcl/c-hcl2).

100% function coverage, ~99% line coverage, exercised under AddressSanitizer and
allocation fault-injection.
