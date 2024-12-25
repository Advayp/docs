---
title: 'init'
sidebar_label: 'init'
pagination_prev: null
pagination_next: null
---

Initialize a new Spice app in the current working directory.

### Usage

```shell
spice init [app_name] [flags]
```

#### Arguments

- `app_name` An optional argument specifying the name of app

#### Flags

- `-h`, `--help` Print this help message

### Examples:

```shell
spice init
```

If called without `app_name`, Spice will prompt for a name and create the Spicepod in the current working directory.

### Additional Example

**Command**:

```shell
spice init <app_name>
```

**Output**:

```
2024/12/18 16:06:21 INFO Initialized <app_name>/spicepod.yaml
```

The new Spicepod is populated, by default, with the following configuration:

```yaml
version: v1beta1
kind: Spicepod
name: <app_name>
```
