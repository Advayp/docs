---
title: 'Data Accelerators'
sidebar_label: 'Data Accelerators'
description: ''
sidebar_position: 2
pagination_prev: null
pagination_next: null
---

Data sourced by Data Connectors can be locally materialized and accelerated using a Data Accelerator.

A Data Accelerator will query/fetch data from a connected data source and store/update it locally in an embedded acceleration engine, such as DuckDB or SQLite. To set data refresh behavior, such as refreshing data on an interval see [Data Refresh](./data-refresh.md).

Dataset acceleration is enabled by setting the acceleration configuration. E.g.

```yaml
datasets:
  - name: accelerated_dataset
    acceleration:
      enabled: true
```

For the complete reference specification see [datasets](/reference/spicepod/datasets.md).

By default, datasets will be locally materialized using in-memory Arrow records.

A choice of DuckDB, SQLite, or PostgreSQL engines can be used to materialize data, in-memory, on disk, or in attached databases.

Supported Data Accelerators include:

| Engine Name                       | Description             | Status | Engine Modes     |
| --------------------------------- | ----------------------- | ------ | ---------------- |
| `arrow`                           | In-Memory Arrow Records | Alpha  | `memory`         |
| [`duckdb`](./duckdb.md)           | Embedded DuckDB         | Alpha  | `memory`, `file` |
| [`sqlite`](./sqlite.md)           | Embedded SQLite         | Alpha  | `memory`, `file` |
| [`postgres`](./postgres/index.md) | Attached PostgreSQL     | Alpha  |                  |

## Data Types

Data Accelerators may not support all possible Apache Arrow data types. For complete compatibility, see [specifications](/reference/datatypes.md).

## Data Accelerator Docs

import DocCardList from '@theme/DocCardList';

<DocCardList />