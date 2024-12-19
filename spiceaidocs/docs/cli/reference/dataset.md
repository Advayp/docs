---
title: "dataset"
sidebar_label: "dataset"
pagination_prev: null
pagination_next: null
---

Dataset operations

### Usage

```shell
spice dataset [command]
```

Available `command`s:

- `configure`: Create/configure a dataset directly from the command-line, including customizing components such as remote source (`from` in a Spicepod) and acceleration along with metadata such as name and description.

#### Flags

- `-h`, `--help` Print this help message

### Sample Output

#### Output from Configure

```bash
2024/12/18 01:06:32 INFO dataset name: sample-project
remote-source # Input 1: Name of dataset
2024/12/18 01:06:59 WARN Dataset names with hyphens should be quoted in queries:
i.e. SELECT * FROM "remote-source"
description: description of remote-source # Input 2: Description
from: remote-source-location # Input 3: Source
2024/12/18 01:07:25 INFO locally accelerate (y/n)? (y)
n # Input 4: Acceleration
2024/12/18 01:07:32 INFO Saved datasets/remote-source/dataset.yaml
```
