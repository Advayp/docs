---
title: "connect"
sidebar_label: "connect"
pagination_prev: null
pagination_next: null
---

Connect to an app on Spice's Cloud Platform.

### Requirements

- Authentication to Spice's Cloud Platform via [`login`](/cli/reference/login)

### Usage

```shell
spice connect [path] [flags]
```

- `path`: Path to project.

#### Flags

- `-h`, `--help` Print this help message

### Examples

```shell
spice connect spiceai/quickstart
```

```shell
spice connect spiceai/tpch
```
