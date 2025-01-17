---
title: 'Databricks Catalog Connector'
sidebar_label: 'Databricks'
description: 'Connect to a Databricks Unity Catalog provider.'
sidebar_position: 1
pagination_prev: null
pagination_next: null
---

Connect to a [Databricks Unity Catalog](https://www.databricks.com/product/unity-catalog) as a catalog provider for federated SQL query using [Spark Connect](https://www.databricks.com/blog/2022/07/07/introducing-spark-connect-the-power-of-apache-spark-everywhere.html) or directly from [Delta Lake](https://delta.io/) tables.

## Configuration

```yaml
catalogs:
  - from: databricks:my_uc_catalog
    name: uc_catalog # tables from this catalog will be available in the "uc_catalog" catalog in Spice
    include:
      - "*.my_table_name" # include only the "my_table_name" tables
    params:
      mode: delta_lake # or spark_connect
      databricks_endpoint: dbc-a12cd3e4-56f7.cloud.databricks.com
    dataset_params:
      # delta_lake S3 parameters
      databricks_aws_region: us-west-2
      databricks_aws_access_key_id: ${secrets:aws_access_key_id}
      databricks_aws_secret_access_key: ${secrets:aws_secret_access_key}
      databricks_aws_endpoint: s3.us-west-2.amazonaws.com
      # spark_connect parameters
      databricks_cluster_id: 1234-567890-abcde123
```

## `from`

The `from` field is used to specify the catalog provider. For Databricks, use `databricks:<catalog_name>`. The `catalog_name` is the name of the catalog in the Databricks Unity Catalog you want to connect to.

## `name`

The `name` field is used to specify the name of the catalog in Spice. Tables from the Databricks catalog will be available in the schema with this name in Spice. The schema hierarchy of the external catalog is preserved in Spice.

## `include`

Use the `include` field to specify which tables to include from the catalog. The `include` field supports glob patterns to match multiple tables. For example, `*.my_table_name` would include all tables with the name `my_table_name` in the catalog from any schema. Multiple `include` patterns are OR'ed together and can be specified to include multiple tables.

## `params`

The `params` field is used to configure the connection to the Databricks Unity Catalog. The following parameters are supported:

- `mode`: The execution mode for querying against Databricks. The default is `spark_connect`. Possible values:
  - `spark_connect`: Use Spark Connect to query against Databricks. Requires a Spark cluster to be available.
  - `delta_lake`: Query directly from Delta Tables. Requires the object store credentials to be provided.
- `databricks_endpoint`: The Databricks workspace endpoint, e.g. `dbc-a12cd3e4-56f7.cloud.databricks.com`.
- `databricks_token`: The Databricks API token to authenticate with the Unity Catalog API. Use the [secret replacement syntax](../secret-stores/index.md) to reference a secret, e.g. `${secrets:my_databricks_token}`.
- `databricks_use_ssl`: If true, use a TLS connection to connect to the Databricks endpoint. Default is `true`.

## `dataset_params`

The `dataset_params` field is used to configure the dataset-specific parameters for the catalog. The following parameters are supported:

### Spark Connect parameters

- `databricks_cluster_id`: The ID of the compute cluster in Databricks to use for the query. e.g. `1234-567890-abcde123`.

### Delta Lake object store parameters

Configure the connection to the object store when using `mode: delta_lake`. Use the [secret replacement syntax](../secret-stores/index.md) to reference a secret, e.g. `${secrets:aws_access_key_id}`.

#### AWS S3

- `databricks_aws_region`: The AWS region for the S3 object store. E.g. `us-west-2`.
- `databricks_aws_access_key_id`: The access key ID for the S3 object store. 
- `databricks_aws_secret_access_key`: The secret access key for the S3 object store.
- `databricks_aws_endpoint`: The endpoint for the S3 object store. E.g. `s3.us-west-2.amazonaws.com`.

Example:
  
```yaml
catalogs:
  - from: databricks:my_uc_catalog
    name: uc_catalog
    include:
      - "*.my_table_name"
    params:
      mode: delta_lake
      databricks_endpoint: dbc-a12cd3e4-56f7.cloud.databricks.com
    dataset_params:
      databricks_aws_region: us-west-2
      databricks_aws_access_key_id: ${secrets:aws_access_key_id}
      databricks_aws_secret_access_key: ${secrets:aws_secret_access_key}
      databricks_aws_endpoint: s3.us-west-2.amazonaws.com
```

#### Azure Blob

:::info Note
One of the following auth values must be provided for Azure Blob:

- `databricks_azure_storage_account_key`, 
- `databricks_azure_storage_client_id` and `azure_storage_client_secret`, or 
- `databricks_azure_storage_sas_key`.
:::

- `databricks_azure_storage_account_name`: The Azure Storage account name.
- `databricks_azure_storage_account_key`: The Azure Storage master key for accessing the storage account.
- `databricks_azure_storage_client_id`: The service principal client id for accessing the storage account.
- `databricks_azure_storage_client_secret`: The service principal client secret for accessing the storage account.
- `databricks_azure_storage_sas_key`: The shared access signature key for accessing the storage account.
- `databricks_azure_storage_endpoint`: The endpoint for the Azure Blob storage account.

Example:
  
```yaml
catalogs:
  - from: databricks:my_uc_catalog
    name: uc_catalog
    include:
      - "*.my_table_name"
    params:
      mode: delta_lake
      databricks_endpoint: dbc-a12cd3e4-56f7.cloud.databricks.com
    dataset_params:
      databricks_azure_storage_account_name: myaccount
      databricks_azure_storage_account_key: ${secrets:azure_storage_account_key}
      databricks_azure_storage_endpoint: myaccount.blob.core.windows.net
```

#### Google Storage (GCS)

- `google_service_account`: Filesystem path to the Google service account JSON key file.

Example:
  
```yaml
catalogs:
  - from: databricks:my_uc_catalog
    name: uc_catalog
    include:
      - "*.my_table_name"
    params:
      mode: delta_lake
      databricks_endpoint: dbc-a12cd3e4-56f7.cloud.databricks.com
    dataset_params:
      databricks_google_service_account: /path/to/service-account.json
```