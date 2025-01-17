---
title: 'Snowflake Data Connector'
sidebar_label: 'Snowflake Data Connector'
description: 'Snowflake Data Connector Documentation'
pagination_prev: null
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

The Snowflake Data Connector enables federated SQL queries across datasets in the [Snowflake Cloud Data Warehouse](https://www.snowflake.com/).

```yaml
datasets:
  - from: snowflake:DATABASE.SCHEMA.TABLE
    name: table
    params:
      snowflake_warehouse: COMPUTE_WH
      snowflake_role: accountadmin
```

:::info[Hint]
Unquoted table identifiers should be UPPERCASED in the `from` field. See [Identifier resolution](https://docs.snowflake.com/en/sql-reference/identifiers-syntax#label-identifier-casing).
:::

### Parameters

- `from`: a Snowflake fully qualified table name (database.schema.table). For instance `snowflake:SNOWFLAKE_SAMPLE_DATA.TPCH_SF1.LINEITEM` or `snowflake:TAXI_DATA."2024".TAXI_TRIPS`
- `snowflake_warehouse`: optional, specifies the [Snowflake Warehouse](https://docs.snowflake.com/en/user-guide/warehouses-tasks) to use
- `snowflake_role`: optional, specifies the role to use for accessing Snowflake data

### Auth

The connector supports password-based and [key-pair](https://docs.snowflake.com/en/user-guide/key-pair-auth) authentication that must be configured using `spice login snowflake` or using [Secrets Stores](/components/secret-stores). Login requires the account identifier ('orgname-accountname' format) - use [Finding the organization and account name for an account](https://docs.snowflake.com/en/user-guide/admin-account-identifier#finding-the-organization-and-account-name-for-an-account) instructions.

<img width="800" src="/img/snowflake/ui-snowsight-account-identifier.png" />

<Tabs>
  <TabItem value="env" label="Env">

    ```bash
    # Password-based
    SPICE_SECRET_SNOWFLAKE_ACCOUNT=<account-identifier> \
    SPICE_SECRET_SNOWFLAKE_USERNAME=<username> \
    SPICE_SECRET_SNOWFLAKE_PASSWORD=<password> \
    spice run
    # Key-pair (the `<private-key-passphrase>` is an optional parameter and is used for encrypted private key only)
    SPICE_SECRET_SNOWFLAKE_ACCOUNT=<account-identifier> \
    SPICE_SECRET_SNOWFLAKE_USERNAME=<username> \
    SPICE_SECRET_SNOWFLAKE_SNOWFLAKE_PRIVATE_KEY_PATH=<path-to-private-key> \
    SPICE_SECRET_SNOWFLAKE_SNOWFLAKE_PRIVATE_KEY_PASSPHRASE=<private-key-passphrase> \
    spice run
    ```

    or using the Spice CLI:

    ```bash
    # Password-based
    spice login snowflake -a <account-identifier> -u <username> -p <password>
    # Key-pair (the `<private-key-passphrase>` is an optional parameter and is used for encrypted private key only)
    spice login snowflake -a <account-identifier> -u <username> -k <path-to-private-key> -s <private-key-passphrase>
    ```

    The CLI will create or update an `.env` file that looks like:
    ```bash
    SPICE_SNOWFLAKE_ACCOUNT="account"
    SPICE_SNOWFLAKE_PASSWORD="pass"
    SPICE_SNOWFLAKE_USERNAME="user"
    ```

    Configure the spicepod to load secrets from the `env` secret store: (Note: This is the default setting)

    `spicepod.yaml`
    ```yaml
    version: v1beta1
    kind: Spicepod
    name: spice-app

    secrets:
      - from: env
        name: env

    datasets:
      - from: snowflake:DATABASE.SCHEMA.TABLE
        name: table
        params:
          snowflake_warehouse: COMPUTE_WH
          snowflake_role: accountadmin
          snowflake_username: ${env:SPICE_SNOWFLAKE_USERNAME}
          snowflake_password: ${env:SPICE_SNOWFLAKE_PASSWORD}
          snowflake_account: ${env:SPICE_SNOWFLAKE_ACCOUNT}
    ```

    Learn more about [Env Secret Store](/components/secret-stores/env).

  </TabItem>
  <TabItem value="k8s" label="Kubernetes">
  
    ```bash
    # Password-based
    kubectl create secret generic snowflake \
      --from-literal=account='<account-identifier>' \
      --from-literal=username='<username>' \
      --from-literal=password='<password>'

    # Key-pair (the `<private-key-passphrase>` is an optional parameter and is used for encrypted private key only)
    kubectl create secret generic snowflake \
      --from-literal=account='<account-identifier>' \
      --from-literal=username='<username>' \
      --from-literal=snowflake_private_key_path='<path-to-private-key>' \
      --from-literal=snowflake_private_key_passphrase='<private-key-passphrase>'
    ```

    `spicepod.yaml`
    ```yaml
    version: v1beta1
    kind: Spicepod
    name: spice-app

    secrets:
      - from: kubernetes:snowflake
        name: snowflake

    datasets:
      - from: snowflake:DATABASE.SCHEMA.TABLE
        name: table
        params:
          snowflake_warehouse: COMPUTE_WH
          snowflake_role: accountadmin
          snowflake_username: ${snowflake.username}
          snowflake_password: ${snowflake.password}
          snowflake_account: ${snowflake.account}
    ```

    Learn more about [Kubernetes Secret Store](/components/secret-stores/kubernetes).

  </TabItem>
  <TabItem value="keyring" label="Keyring">
    Add new keychain entries (macOS) for user and password:

    ```bash
    # Password-based
    security add-generic-password -l "Snowflake Secret" \
    -a spiced -s spice_snowflake_password\
    -w <password>

    # Key-pair (the `<private-key-passphrase>` is an optional parameter and is used for encrypted private key only)
    security add-generic-password -l "Snowflake Secret" \
    -a spiced -s spice_snowflake_snowflake_private_key_path\
    -w $(echo -n '<path-to-private-key>' | base64)
    ```

    `spicepod.yaml`
    ```yaml
    version: v1beta1
    kind: Spicepod
    name: spice-app

    secrets:
      - from: keyring
        name: keyring

    datasets:
      - from: snowflake:DATABASE.SCHEMA.TABLE
        name: table
        params:
          snowflake_warehouse: COMPUTE_WH
          snowflake_role: accountadmin
          snowflake_username: user_name
          snowflake_password: ${keyring:spice_snowflake_password}
          snowflake_account: account_identifier
    ```

    Learn more about [Keyring Secret Store](/components/secret-stores/keyring).

  </TabItem>
</Tabs>

## Example

```yaml
datasets:
  - from: snowflake:SNOWFLAKE_SAMPLE_DATA.TPCH_SF1.LINEITEM
    name: lineitem
    params:
      snowflake_warehouse: COMPUTE_WH
      snowflake_role: accountadmin
```

:::warning[Limitations]

1. Account identifier does not support the [Legacy account locator in a region format](https://docs.snowflake.com/en/user-guide/admin-account-identifier#format-2-legacy-account-locator-in-a-region). Use [Snowflake preferred name in organization format](https://docs.snowflake.com/en/user-guide/admin-account-identifier#format-1-preferred-account-name-in-your-organization).
1. The connector supports password-based and [key-pair](https://docs.snowflake.com/en/user-guide/key-pair-auth) authentication.

:::
