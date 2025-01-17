---
title: "status"
sidebar_label: "status"
pagination_prev: null
pagination_next: null
---
Spice runtime status

### Usage

```shell
spice status [flags]
```

#### Flags

- `--tls-root-certificate-file`   The path to the root certificate file used to verify the Spice.ai runtime server certificate
- `-h`, `--help`   help for status

### Examples

```shell 
>>> spice status

NAME          ENDPOINT        STATUS
http          127.0.0.1:8090  Ready
flight        127.0.0.1:50051 Ready
metrics       N/A             Disabled
opentelemetry 127.0.0.1:50052 Ready
```

### Additional Example

```shell
>>> spice status --tls-root-certificate-file /path/to/cert.pem

NAME          ENDPOINT        STATUS
http          127.0.0.1:8090  Ready
flight        127.0.0.1:50051 Ready
metrics       N/A             Disabled
opentelemetry 127.0.0.1:50052 Ready
```
