---
title: "add"
sidebar_label: "add"
pagination_prev: null
pagination_next: null
---

Adds a Spicepod to the project from a given source.

### Usage

```shell
spice add [source] [flags]
```

- `source`: Location of the Spicepod.

#### Flags

- `-h`, `--help` Print this help message

### Examples

Adding a Spicepod from a source currently using Spice (like `spiceai/quickstart`):

```shell
spice add spiceai/quickstart
```

This adds the following Spicepod under `./spicepods`:

```yaml
version: v1beta1
kind: Spicepod
name: quickstart

datasets:
  - from: s3://spiceai-demo-datasets/taxi_trips/2024/
    name: taxi_trips
    description: taxi trips in s3
    params:
      file_format: parquet
    acceleration:
      enabled: true
```
