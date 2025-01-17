---
title: "models"
sidebar_label: "models"
pagination_prev: null
pagination_next: null
---

Lists models loaded by the Spice runtime

### Usage

```shell 
spice models [flags]
```

#### Flags

- `--tls-root-certificate-file`   The path to the root certificate file used to verify the Spice.ai runtime server certificate
- `-h`, `--help`   help for models

### Examples

```shell
>>> spice models

NAME  FROM                                    DATASETS STATUS
modlz file:/Users/jeadie/Downloads/model.onnx []       Ready
```

### Additional Example

```shell
>>> spice models --tls-root-certificate-file /path/to/cert.pem

NAME  FROM                                    DATASETS STATUS
modlz file:/Users/jeadie/Downloads/model.onnx []       Ready
```
