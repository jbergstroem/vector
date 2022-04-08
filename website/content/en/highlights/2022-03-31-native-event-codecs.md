---
date: "2022-03-31"
title: "New `native` and `native_json` codecs"
description: ""
authors: ["lukesteensen"]
pr_numbers: []
release: "0.21.0"
hide_on_release_notes: false
badges:
  type: "announcement"
---

We have added new experimental `native` and `native_json` codecs that allow
users to provide Vector with data directly in its native format. This allows
simpler, more efficient configuration of some often-requested use cases.

For example, an `exec` source that periodically runs a command can now directly
provide Vector with metrics instead of requiring that they pass through
a `log_to_metric` transform:

```toml
[sources.in]
type = "exec"
mode = "scheduled"
command = ["./scrape.sh"]
decoding.codec = "native_json"
```

```bash
#!/usr/bin/env bash

echo $RANDOM | jq --raw-input --compact-output \
'{
  metric: {
    name: "my_metric",
    counter: {
      value: .|tonumber
    },
    kind: "incremental",
  }
}'
```

The specific JSON schema here is subject to change, but we will be publishing
more thorough guidance as the feature matures. One current limitation of the
JSON-based native codec is that timestamp fields within log messages will be
converted to strings. This is partially due to limitations with JSON itself, but
we are exploring workarounds.

Another example use case is Vector-to-Vector communication via a transport other
than our existing gRPC-based `vector` source and sink. Using the `native`
encoding on a source/sink pair like `kafka` will utilize the same Protocol
Buffers-based encoding scheme, providing compact binary messages that
deserialize directly to the same native representation within Vector.