---
title: 'dataset'
sidebar_label: 'dataset'
pagination_prev: null
pagination_next: null
---

Dataset operations

### Usage

```shell
spice dataset [command]
```

Available `command`s:

- `configure`: Create/configure a dataset directly from the command-line, including customizing components such as Data Connector (`from` in a Spicepod) and acceleration along with metadata.

#### Flags

- `-h`, `--help` Print this help message

### Sample Output

#### Output from Configure

```bash
2024/12/18 01:06:32 INFO dataset name: sample-project
taxi_trips # Input 1: Name of dataset
2024/12/18 01:06:59 WARN Dataset names with hyphens should be quoted in queries:
i.e. SELECT * FROM "remote-source"
description: Taxi trips in s3 # Input 2: Description
from: s3://spiceai-demo-datasets/taxi_trips/2024/  # Input 3: Source
2024/12/18 01:07:25 INFO locally accelerate (y/n)? (y)
n # Input 4: Acceleration
2024/12/18 01:07:32 INFO Saved datasets/remote-source/dataset.yaml
```
