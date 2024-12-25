---
title: 'login'
sidebar_label: 'login'
pagination_prev: null
pagination_next: null
---

Login to the Spice.ai Platform, or other services with sub-commands.

### Usage

```shell
spice login [command] [flags]
```

### Flags

- `-h`, `--help` Print this help message
- `-k`, `--key` string API key (for spice.ai)

#### Available Commands To Connect to Other Services

- `abfs` Login to a Azure Storage Account
- `databricks` Login to a Databricks instance
- `delta_lake` Configure credentials to access a Delta Lake table
- `dremio` Login to a Dremio instance
- `postgres` Login to a Postgres instance
- `s3` Login to an s3 storage
- `sharepoint` Login to a Microsoft 365 sharepoint account
- `snowflake` Login to a Snowflake warehouse
- `spark` Login to a Spark Connect remote

### Examples

```shell
spice login
```

#### Additional Example

```shell
spice login --key <API_KEY>
```

### Examples for Other Services

### Postgres

**Example 1**:

```shell
spice login postgres --password <password>
```

**Example 2**:

```shell
spice login postgres -p <password>
```

**Required Flags**:

- `-p`, `--password` Password to your postgres user

**Sample Output**:

```
2024/12/18 13:59:10 INFO Successfully logged in to Pg
```
