---
title: 'Data Refresh'
sidebar_label: 'Data Refresh'
description: 'Data refresh for accelerated datasets'
sidebar_position: 1
pagination_prev: null
pagination_next: null
---

## Refresh Modes

Spice supports three modes to refresh/update local data from a connected data source. `full` is the default mode.

| Mode      | Description                                          | Example                                                          |
| --------- | ---------------------------------------------------- | ---------------------------------------------------------------- |
| `full`    | Replace/overwrite the entire dataset on each refresh | A table of users                                                 |
| `append`  | Append/add data to the dataset on each refresh       | Append-only, immutable datasets, such as time-series or log data |
| `changes` | Apply incremental changes                            | Customer order lifecycle table                                   |

Example:

```yaml
datasets:
  - from: databricks:my_dataset
    name: accelerated_dataset
    acceleration:
      refresh_mode: full
      refresh_check_interval: 10m
```

### Append

Using `refresh_mode: append` requires the use of a [`time_column` dataset parameter](/reference/spicepod/datasets.md#time_column), specifying a column to compare the local acceleration against the remote source. Data will be incrementally refreshed where the `time_column` value in the remote source is greater-than (gt) the `max(time_column)` value in the local acceleration.

E.g.

```yaml
datasets:
  - from: databricks:my_dataset
    name: accelerated_dataset
    time_column: created_at
    acceleration:
      refresh_mode: append
      refresh_check_interval: 10m
```

If late arriving data or clock-skew needs to be accounted for, an optional overlap can also be specified. See [`acceleration.refresh_append_overlap`](/reference/spicepod/datasets#accelerationrefresh_append_overlap).

### Changes (CDC)

Datasets configured with acceleration `refresh_mode: changes` requires a [Change Data Capture (CDC)](/features/cdc/index.md) supported data connector. Initial CDC support in Spice is supported by the [Debezium data connector](/components/data-connectors/debezium.md).

## Ready State

|                             |           |
| --------------------------- | --------- |
| Supported in `refresh_mode` | Any       |
| Required                    | No        |
| Default Value               | `on_load` |

By default, Spice will return an error for queries against an accelerated dataset that is still loading its initial data. The endpoint [`/v1/ready`](/api/http/ready) is used in production deployments to control when queries are sent to the Spice runtime.

The ready state for an accelerated dataset can be configured using the [`ready_state`](/reference/spicepod/datasets#ready_state) parameter in the dataset configuration.

- `ready_state: on_load`: Default. The dataset is considered ready after the initial load of the accelerated data. For file-based accelerated datasets that have existing data, this will be ready immediately. Queries against this dataset before the data is loaded will return an error.
- `ready_state: on_registration`: The dataset is considered ready when the dataset is registered in Spice, even before the initial data is loaded. Queries against this dataset before the data is loaded will automatically fallback to the federated source. Once the data is loaded, queries will be served from the acceleration.

Example:

```yaml
datasets:
  - from: s3://my_bucket/my_dataset
    name: my_dataset
    ready_state: on_load # or on_registration
    acceleration:
      enabled: true
```

## Filtered Refresh

Typically only a working subset of an entire dataset is used in an application or dashboard. Use these features to filter refresh data, creating a smaller subset for faster processing and to reduce the data transferred and stored locally.

- [Refresh SQL](#refresh-sql) - Specify the filter as arbitrary SQL to be pushed down to the remote source.
- [Refresh Data Window](#refresh-data-window) - Filters data from the remote source outside the specified time window.

### Refresh SQL

|                             |           |
| --------------------------- | --------- |
| Supported in `refresh_mode` | Any       |
| Required                    | No        |
| Default Value               | Unset     |

Refresh SQL supports specifying filters for data accelerated from the connected source using arbitrary SQL.

Filters will be pushed down to the remote source when possible, so only the requested data will be transferred over the network.

Example:

```yaml
datasets:
  - from: databricks:my_dataset
    name: accelerated_dataset
    acceleration:
      enabled: true
      refresh_mode: full
      refresh_check_interval: 10m
      refresh_sql: |
        SELECT * FROM accelerated_dataset WHERE city = 'Seattle'
```

The `refresh_sql` parameter can be updated at runtime on-demand using `PATCH /v1/datasets/:name/acceleration`. This change is temporary and will revert to the `spicepod.yml` definition at the next runtime restart.

Columns can be selected in the query via the `SELECT` clause, but only column names are supported. Arbitrary expressions or aliases are not supported.

Example:

```bash
curl -i -X PATCH \
     -H "Content-Type: application/json" \
     -d '{
           "refresh_sql": "SELECT city, state FROM accelerated_dataset WHERE city = 'Bellevue'"
         }' \
     127.0.0.1:8090/v1/datasets/accelerated_dataset/acceleration
```

Queries that return zero results will fallback to the behavior specified by the [`on_zero_results` parameter](#behavior-on-zero-results), and will not have the `refresh_sql` applied to the results from the fallback. The `refresh_sql` only applies to acceleration refresh tasks.

For the complete reference, view the `refresh_sql` section of [datasets](/reference/spicepod/datasets.md#accelerationrefresh_sql).

:::warning[Limitations]

- When `refresh_mode: changes` is specified, Refresh SQL can only modify the selected columns and cannot apply filters.
- Running queries while using refresh SQL will not fallback to the source if any query returns more than zero rows, even when querying against columns that are not explicitly filtered by the refresh SQL. This may result in queries returning partial data, depending on the filters applied in the refresh SQL.
- Refresh SQL only supports filtering data from the current dataset - joining across other datasets is not supported.
- Refresh SQL modifications made via API are temporary and will revert after a runtime restart.

:::

### Refresh Data Window

|                             |                  |
| --------------------------- | ---------------- |
| Supported in `refresh_mode` | `full`, `append` |
| Required                    | No               |
| Default Value               | Unset            |

The `refresh_data_window` parameter supports refreshing data that falls within the specified time window. The `refresh_data_window` is applied cumulatively to any filters specified by the [`refresh_sql`](#refresh-sql), and applies a time filter based on `now() - refresh_data_window`. For example, the following configuration:

```yaml
time_column: column_time
acceleration:
  refresh_sql: "SELECT * FROM my_dataset WHERE column_one = 'value'"
  refresh_data_window: 1d
```

In this example, `refresh_data_window` is converted into an effective Refresh SQL of `SELECT * FROM my_dataset WHERE column_one = 'value' AND column_time > (now() - interval '1' day)`. The `time_column` column can be specified in the `refresh_sql` in conjunction with the `refresh_data_window`, and both filters are combined with `AND`.

This parameter relies on the `time_column` dataset parameter specifying a column that is a timestamp type. Optionally, the `time_format` can be specified to instruct the Spice runtime on how to interpret timestamps in the `time_column`.

*Example with `refresh_sql`:*

```yaml
datasets:
  - from: databricks:my_dataset
    name: accelerated_dataset
    time_column: created_at
    acceleration:
      enabled: true
      refresh_mode: full
      refresh_check_interval: 10m
      refresh_sql: |
        SELECT * FROM accelerated_dataset WHERE city = 'Seattle'
      refresh_data_window: 1d
```

This example will only accelerate data from the federated source that matches the filter `city = 'Seattle'` and is less than 1 day old.

*Example with `on_zero_results`:*

```yaml
datasets:
  - from: databricks:my_dataset
    name: accelerated_dataset
    time_column: created_at
    acceleration:
      enabled: true
      refresh_mode: full
      refresh_check_interval: 10m
      refresh_sql: |
        SELECT * FROM accelerated_dataset WHERE city = 'Seattle'
      refresh_data_window: 1d
      on_zero_results: use_source
```

This example will only accelerate data from the federated source that matches the filter `city = 'Seattle'` and is less than 1 day old. If a query against the accelerated data returns zero results, the query will fallback to the source and return the direct results without any filtering.

If a query against the accelerated data returns some results, the query will not fall back. For example, attempting to query for the last 2 days of data would only return the last 1 day of data without falling back.

## Behavior on Zero Results

|                             |                  |
| --------------------------- | ---------------- |
| Supported in `refresh_mode` | `full`, `append` |
| Required                    | No               |
| Default Value               | `return_empty`   |

By default, accelerated datasets only return locally materialized data. If this local data is a subset of the full dataset in the federated source—due to settings like `refresh_sql`, `refresh_data_window`, or retention policies—queries against the accelerated dataset may return zero results, even when the federated table would return results.

To address this, `on_zero_results: use_source` can be configured in the acceleration configuration. Queries returning zero results will fall back to the federated source, returning results from querying the underlying data.

`on_zero_results`:

- `return_empty` (Default) - Return an empty result set when no data is found in the accelerated dataset.
- `use_source` - Fall back to querying the federated table when no data is found in the accelerated dataset.

Example:

```yaml
datasets:
  - from: databricks:my_dataset
    name: accelerated_dataset
    acceleration:
      enabled: true
      refresh_sql: SELECT * FROM accelerated_dataset where city = 'Seattle'
      on_zero_results: use_source
```

In this example a query against `accelerated_dataset` within Spice like `SELECT * FROM accelerated_dataset WHERE city = 'Portland'` would initially query against the accelerated data, see that it returns zero results and then fallback to querying against the federated table in Databricks.

:::warning

- It is possible that even though an accelerated table returns some results, it may not contain all the data that would be returned by the federated table. `on_zero_results` only controls the behavior in the simple case where no data is returned by the acceleration for a given query.

:::

## Refresh Interval

|                             |                  |
| --------------------------- | ---------------- |
| Supported in `refresh_mode` | `full`, `append` |
| Required                    | No               |
| Default Value               | Unset            |

The [`refresh_check_interval`](/reference/spicepod/datasets#accelerationrefresh_check_interval) parameter controls how often the accelerated dataset is refreshed.

Example:

```yaml
datasets:
  - from: spice.ai/spiceai/quickstart/datasets/taxi_trips
    name: taxi_trips
    acceleration:
      enabled: true
      refresh_mode: full
      refresh_check_interval: 10s
```

This configuration will refresh `taxi_trips` data every 10 seconds.

## Refresh On-Demand

:::info

Supported for accelerators with a `refresh_mode` of `full` or `append`.

:::

Accelerated datasets can be refreshed on-demand via the `refresh` CLI command or `POST /v1/datasets/:name/acceleration/refresh` API endpoint.

CLI example:

```bash
spice refresh eth_recent_blocks
```

API example using cURL:

```bash
curl -i -XPOST 127.0.0.1:8090/v1/datasets/eth_recent_blocks/acceleration/refresh
```

with response:

```bash
HTTP/1.1 201 Created
content-type: application/json
content-length: 55
date: Thu, 11 Apr 2024 20:11:18 GMT

{"message":"Dataset refresh triggered for eth_recent_blocks."}
```

:::warning[Note]
On-demand refresh always initiates a new refresh, terminating any in-progress refresh for the dataset.
:::

## Refresh Retries

|                                      |                  |
| ------------------------------------ | ---------------- |
| Supported in `refresh_mode`          | `full`, `append` |
| Required                             | No               |
| Default `refresh_retry_enabled`      | `false`          |
| Default `refresh_retry_max_attempts` | Unset            |

By default, data refreshes for accelerated datasets are retried on transient errors (connectivity issues, compute warehouse goes idle, etc.) using a [Fibonacci](https://en.wikipedia.org/wiki/Fibonacci_sequence) backoff strategy.

Retry behavior can be configured using the [`acceleration.refresh_retry_enabled`](/reference/spicepod/datasets#accelerationrefresh_retry_enabled) and [`acceleration.refresh_retry_max_attempts`](/reference/spicepod/datasets#accelerationrefresh_retry_max_attempts) parameters.

Example: Disable retries

```yaml
datasets:
  - from: spice.ai/spiceai/quickstart/datasets/taxi_trips
    name: taxi_trips
    acceleration:
      refresh_retry_enabled: false
      refresh_check_interval: 30s
```

Example: Limit retries to a maximum of 10 attempts

```yaml
datasets:
  - from: spice.ai/spiceai/quickstart/datasets/taxi_trips
    name: taxi_trips
    acceleration:
      refresh_retry_max_attempts: 10
      refresh_check_interval: 30s
```

## Retention Policy

|                                    |                  |
| ---------------------------------- | ---------------- |
| Supported in `refresh_mode`        | `full`, `append` |
| Required                           | No               |
| Default `retention_check_enabled`  | `false`          |
| Default `retention_period`         | Unset            |
| Default `retention_check_interval` | Unset            |

Accelerated datasets can be set to automatically evict time-series data exceeding a retention period by setting a retention policy based on the configured `time_column` and `acceleration.retention_period`.

The policy is set using the [`acceleration.retention_check_enabled`](/reference/spicepod/datasets#accelerationretention_check_enabled), [`acceleration.retention_period`](/reference/spicepod/datasets#accelerationretention_period) and [`acceleration.retention_check_interval`](/reference/spicepod/datasets#accelerationretention_check_interval) parameters, along with the [`time_column`](/reference/spicepod/datasets#time_column) and [`time_format`](/reference/spicepod/datasets#time_format) dataset parameters.

When `retention_check_enabled` is set to `true`, `retention_check_interval` and `retention_period` are required parameters.

## Refresh Jitter

|                                  |                  |
| -------------------------------- | ---------------- |
| Supported in `refresh_mode`      | `full`, `append` |
| Required                         | No               |
| Default `refresh_jitter_enabled` | `false`          |
| Default `refresh_jitter_max`     | Unset            |

Accelerated datasets can include a random jitter in their refresh interval to prevent the [Thundering herd problem](https://en.wikipedia.org/wiki/Thundering_herd_problem), where multiple datasets refresh simultaneously. The jitter is a random value between 0 and `refresh_jitter_max`, which is added to or subtracted from the base `refresh_check_interval`. If `refresh_jitter_max` is not specified, it defaults to 10% of `refresh_check_interval`.

Refresh Jitter applies to the initial dataset load. If multiple similarly configured Spice instances are restarted at the same time, they will load with a jitter between 0 and `refresh_jitter_max`.

Example:

```yaml
datasets:
  - from: spice.ai/spiceai/quickstart/datasets/taxi_trips
    name: taxi_trips
    acceleration:
      refresh_check_interval: 10s
      refresh_jitter_enabled: true
      refresh_jitter_max: 1s
```

In the configuration above:

1. The initial load will include a random delay between **0** and **1 second**.
1. Subsequent refresh intervals will vary randomly between **9 seconds** and **11 seconds**.

Refresh jitter configuration:

- [`refresh_jitter_enabled`](/reference/spicepod/datasets#accelerationrefresh_jitter_enabled)
- [`refresh_jitter_max`](/reference/spicepod/datasets#accelerationrefresh_jitter_max)

## Configuration Examples

### Accelerating a full set of data that sometimes changes

In this example, Spice connects with a dataset that changes infrequently and is not configured for CDC. For example, a list of product categories.

```yaml
datasets:
  - from: mysql:product_categories
    name: product_categories
    acceleration:
      refresh_mode: full
      refresh_check_interval: 8h
```

In this scenario, Spice uses a simple acceleration configuration - full refreshes on an 8 hour schedule. No additional behaviors are enabled, so queries matching for new product codes will return no results until the next refresh cycle.

### Accelerating a subset of data that changes frequently

In this example, Spice connects with a dataset that has frequently changing data that is not configured for CDC. For example, user's posts on a social media platform.

```yaml
datasets:
  - from: mysql:posts
    name: posts
    acceleration:
      refresh_mode: full
      refresh_check_interval: 10m
      refresh_sql: "SELECT * FROM posts WHERE updated_at > now() - interval '1' day"
      on_zero_results: use_source
```

With this configuration, Spice will refresh every 10 minutes accelerating posts that have been updated in the last day.

When querying for posts by direct ID, if a post is not accelerated Spice will fallback to retrieving the post from the non-accelerated source due to the behavior of `on_zero_results: use_source`.

However, if querying for a range of posts that includes some which have updated in the last day Spice will only return those results without falling back to the source. This could result in queries for a range of posts excluding posts that exist in the non-accelerated source because they have been filtered out due to their `updated_at` value.

### Accelerating application logs

In this example, Spice connects to a data source that is immutable, receives new rows, and is not configured for CDC. For example, a database that contains some application logs.

```yaml
datasets:
  - from: duckdb:logs
    name: logs
    time_column: created_at
    params:
      duckdb_open: logs.duckdb
    acceleration:
      refresh_mode: append
      refresh_check_interval: 10m
      refresh_sql: "SELECT * FROM logs WHERE asset = 'asset_id'"
      refresh_data_window: 1d
      on_zero_results: use_source
      retention_check_enabled: true
      retention_period: 7d
      retention_check_interval: 10m
```

This acceleration configuration applies a number of different behaviors:

1. A `refresh_data_window` was specified. When Spice starts, it will apply this `refresh_data_window` to the `refresh_sql`, and retrieve only the last day's worth of logs with an `asset = 'asset_id'`.
2. Because a `refresh_sql` is specified, every refresh (including initial load) will have the filter applied to the refresh query.
3. 10 minutes after loading, as specified by the `refresh_check_interval`, the first refresh will occur - retrieving new rows where `asset = 'asset_id'`.
4. Running a query to retrieve logs with an `asset` that is *not* `asset_id` will fall back to the source, because of the `on_zero_results: use_source` parameter.
5. Running a query to retrieve a log longer than 1 day ago will fall back to the source, because of the `on_zero_results: use_source` parameter.
6. Running a query to retrieve logs within a range of now to longer than 1 day ago will only return logs from the last day. This is due to the `refresh_data_window` only accelerating the last day's worth of logs, which will return some results. Because results are returned, Spice will not fall back to the source even though `on_zero_results: use_source` is specified.
7. Spice will retain newly appended log rows for 7 days before discarding them, as specified by the `retention_*` parameters.
