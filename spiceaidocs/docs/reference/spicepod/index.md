---
title: "Manifest syntax for Spicepods"
sidebar_position: 1
sidebar_label: "Spicepod specification"
description: "Detailed documentation on the Spicepod manifest syntax (spicepod.yaml)"
---

# About YAML syntax for Spicepod manifests (spicepod.yaml)

Spicepod manifests use YAML syntax and must be named `spicepod.yaml` or `spicepod.yml`. If you are new to YAML and want to learn more, see "[Learn YAML in Y minutes](https://learnxinyminutes.com/docs/yaml/)."

Spicepod manifest files are stored in the root directory of your application code.

# Syntax

## version

The version of the Spicepod manifest. The current version is `v1beta1`.

## kind

The specific kind of Spicepod manifest. For a Spicepod, this is obviously `Spicepod`.

## name

The name of the Spicepod.

## secrets

The secrets section in the Spicepod manifest is optional and is used to configure how secrets are stored and accessed by the Spicepod. For more information, see [Secret Stores](/components/secret-stores).

### secrets.from

The `from` field is a string that represents the Uniform Resource Identifier (URI) for the secret store. This URI is composed of two parts: a prefix indicating the Secret Store to use, and an optional selector that specifies the secret to retrieve.

The syntax for the `from` field is as follows:

```yaml
from: <secret_store>:<selector>
```

Where:

- `<secret_store>`: The Secret Store to use. Currently, Spice supports [`env`](/components/secret-stores/env/index.md), [`kubernetes`](/components/secret-stores/kubernetes/index.md), [`keyring`](/components/secret-stores/keyring/index.md), [`aws-secrets-manager`](/components/secret-stores/aws-secrets-manager/index.md). The default value is [`env`](/components/secret-stores/env/index.md).
- `<selector>`: The identifier of the secret within the store to load.

#### Example

```yaml
secrets:
  - from: env
```

### secrets.name

The name of the secret store. This is used to reference the store in the secret replacement syntax, `${<secret_store_name>:<key_name>}`.

#### Example

```yaml
secrets:
  - from: env
  - name: local-environment
```

## runtime

### runtime.num_of_parallel_loading_at_start_up

This configuration setting determines the maximum number of datasets that can be loaded in parallel during startup. This parallel loading capability accelerates Spice's startup process when multiple datasets are configured.

### runtime.results_cache

The results cache section specifies runtime cache configuration. [Learn more](/features/caching).

```yaml
runtime:
  results_cache:
    enabled: true
    cache_max_size: 128MiB
    eviction_policy: lru
    item_ttl: 1s
```

| Name              | Constraint | Description                                               | Default                                                                                                          |
| ----------------- | ---------- | --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `enabled`         | Optional   | Enable the results cache                                  | `true`                                                                                                           |
| `cache_max_size`  | Optional   | The maximum capacity of the cache.                        | `128MiB`                                                                                                         |
| `eviction_policy` | Optional   | The replacement policy of the cache when it's at capacity | [`lru`](https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU) (only supported eviction policy as of now) |
| `item_ttl`        | Optional   | Cache entry expiration time                               | `1s`                                                                                                             |

### runtime.tls

The TLS section specifies the configuration for enabling Transport Layer Security (TLS) for all endpoints exposed by the runtime. [Learn more about enabling TLS](/api/tls).

In addition to configuring TLS via the manifest, TLS can also be configured via `spiced` command line arguments using with `--tls-enabled true` and `--tls-certificate`/`--tls-certificate-file` and `--tls-key`/`--tls-key-file` flags.

#### Additional Components of `runtime.tls`

| Field Name                     | Description                                                                                                             |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| `runtime.tls.enabled`          | Enables or disables TLS for the runtime endpoints                                                                       |
| `runtime.tls.certificate`      | The TLS certificate to use for securing the runtime endpoints. Can also come from [secrets](/components/secret-stores). |
| `runtime.tls.certificate_file` | The path to the TLS PEM-encoded certificate file. Only one of `certificate` and `certificate_file` can be used          |
| `runtime.tls.key`              | The TLS key to use for secruing the runtime endpoints. Can also come from [secrets](/components/secret-stores)          |
| `runtime.tls.key_file`         | Path to the TLS PEM-encoded key file. Only one of `key` or `key_file` can be used.                                      |

#### Example

**With `certificate` and `key`**:

```yaml
runtime:
  tls:
    enabled: true
    certificate: |
      -----BEGIN CERTIFICATE-----
      ...
      -----END CERTIFICATE-----
    key: |
      -----BEGIN PRIVATE KEY-----
      ...
      -----END PRIVATE KEY-----
```

**With `certificate_file` and `key_file`**:

```yaml
runtime:
  tls:
    enabled: true
    certificate_file: /path/to/certificate.pem
    key_file: /path/to/key.pem
```

### runtime.task_history

The task history section specifies runtime task history configuration.

```yaml
runtime:
  task_history:
    enabled: true
    captured_output: none
    retention_period: 8h
    retention_check_interval: 15m
```

| Field Name                 | Description                                                          | Constraint | Default                                   |
| -------------------------- | -------------------------------------------------------------------- | ---------- | ----------------------------------------- |
| `enabled`                  | Whether or not task history is enabled                               | Optional   | `true`                                    |
| `captured_output`          | What level of output is captured by the task history table           | Optional   | `none`. Other values include `truncated`. |
| `retention_period`         | Duration of retention for recrods captured in the task history table | Optional   | `8h`, or 8 hours                          |
| `retention_check_interval` | Frequency of checking records for removal                            | Optional   | `15m`, or 15 minutes                      |

### runtime.cors

The CORS section specifies the configuration for enabling Cross-Origin Resource Sharing (CORS) for the HTTP endpoint. By default, CORS is disabled.

| Field Name                    | Description                                    | Default                           |
| ----------------------------- | ---------------------------------------------- | --------------------------------- |
| `runtime.cors.enabled`        | Enables or disables CORS for the HTTP endpoint | `false`                           |
| `runtime.cors.allowed_origins | List of allowed origins for CORS requests      | `["*"]`, which allows all origins |

**Example**:

```yaml
runtime:
  cors:
    enabled: true
    allowed_origins: ["https://example.com"]
```

This configuration allows requests from the `https://example.com` origin only.

## metadata

An optional `map` of metadata.

**Example**

```yaml
metadata:
  epoch_time: 1605312000
  period: 72h
  interval: 1m
  granularity: 10s
  episodes: 10
```

## datasets

A Spicepod can contain one or more [datasets](./datasets.md) referenced by relative path.

**Example**

A datasets referenced by relative path.

```yaml
datasets:
  - ref: datasets/uniswap_v2_eth_usdc
```

A dataset defined inline.

```yaml
datasets:
  - from: spice.ai/spiceai/quickstart/datasets/taxi_trips
    name: taxi_trips
    acceleration:
      enabled: true
      refresh_mode: full
      refresh_check_interval: 1h
```

Learn more about the specific syntax and configuration settings [here](./datasets.md).

## models

A Spicepod can contain one or more [models](./models.md) referenced by relative path.

**Example**

A model referenced by path.

```yaml
models:
  - from: models/drive_stats
```

A model defined inline.

```yaml
models:
  - from: spiceai/lukekim/smart/models/drive_stats:latest
    name: drive_stats
    datasets:
      - drive_stats_inferencing
```

Learn more about the specific syntax and configuration settings [here](./models.md).

## embeddings

A Spicepod can contain one or more [embeddings](./embeddings.md) referenced by relative path.

**Example**

An embeddings model referenced by path.

```yaml
embeddings:
  - from: embeddings/openai_text_embedding_3
```

An embedding defined inline.

```yaml
embeddings:
  - name: hf_baai_bge
    from: huggingface:huggingface.co/BAAI/bge-small-en-v1.5
```

Learn more about the specific syntax and configuration settings [here](./embeddings.md).

## dependencies

A list of dependent Spicepods.

```yaml
dependencies:
  - lukekim/demo
  - spicehq/nfts
```

## views

A Spicepod can contain one or more views which are virtual tables defined by SQL queries.

**Example**

```yaml
views:
  - name: rankings
    sql: |
      WITH a AS (
        SELECT products.id, SUM(count) AS count
        FROM orders
        INNER JOIN products ON orders.product_id = products.id
        GROUP BY products.id
      )
      SELECT name, count
      FROM products
      LEFT JOIN a ON products.id = a.id
      ORDER BY count DESC
      LIMIT 5
```
