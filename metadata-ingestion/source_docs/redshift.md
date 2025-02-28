
# Redshift

For context on getting started with ingestion, check out our [metadata ingestion guide](../README.md).

## Setup

To install this plugin, run `pip install 'acryl-datahub[redshift]'`.

## Capabilities

This plugin extracts the following:

- Metadata for databases, schemas, views and tables
- Column types associated with each table
- Also supports PostGIS extensions
- Table, row, and column statistics via optional [SQL profiling](./sql_profiles.md)
- Table lineage

## Quickstart recipe

Check out the following recipe to get started with ingestion! See [below](#config-details) for full configuration options.

For general pointers on writing and running a recipe, see our [main recipe guide](../README.md#recipes).

```yml
source:
  type: redshift
  config:
    # Coordinates
    host_port: example.something.us-west-2.redshift.amazonaws.com:5439
    database: DemoDatabase

    # Credentials
    username: user
    password: pass

    # Options
    options:
      # driver_option: some-option

    include_views: True # whether to include views, defaults to True
    include_tables: True # whether to include views, defaults to True

sink:
  # sink configs
```

<details>
  <summary>Extra options when running Redshift behind a proxy</summary>

This requires you to have already installed the Microsoft ODBC Driver for SQL Server.
See https://docs.microsoft.com/en-us/sql/connect/python/pyodbc/step-1-configure-development-environment-for-pyodbc-python-development?view=sql-server-ver15

```yml
source:
  type: redshift
  config:
    host_port: my-proxy-hostname:5439

    options:
      connect_args:
        sslmode: "prefer" # or "require" or "verify-ca"
        sslrootcert: ~ # needed to unpin the AWS Redshift certificate

sink:
  # sink configs
```

</details>

## Config details

Note that a `.` is used to denote nested fields in the YAML recipe.

As a SQL-based service, the Athena integration is also supported by our SQL profiler. See [here](./sql_profiles.md) for more details on configuration.

| Field                       | Required | Default            | Description                                                                                                                                                                             |
|-----------------------------| -------- |--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `username`                  |          |                    | Redshift username.                                                                                                                                                                      |
| `password`                  |          |                    | Redshift password.                                                                                                                                                                      |
| `host_port`                 | ✅       |                    | Redshift host URL.                                                                                                                                                                      |
| `database`                  |          |                    | Redshift database.                                                                                                                                                                      |
| `database_alias`            |          |                    | Alias to apply to database when ingesting.                                                                                                                                              |
| `env`                       |          | `"PROD"`           | Environment to use in namespace when constructing URNs.                                                                                                                                 |
| `options.<option>`          |          |                    | Any options specified here will be passed to SQLAlchemy's `create_engine` as kwargs.<br />See https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create_engine for details. |
| `table_pattern.allow`       |          |                    | List of regex patterns for tables to include in ingestion.                                                                                                                              |
| `table_pattern.deny`        |          |                    | List of regex patterns for tables to exclude from ingestion.                                                                                                                            |
| `table_pattern.ignoreCase`  |          | `True`             | Whether to ignore case sensitivity during pattern matching.                                                                                                                             |
| `schema_pattern.allow`      |          |                    | List of regex patterns for schemas to include in ingestion.                                                                                                                             |
| `schema_pattern.deny`       |          |                    | List of regex patterns for schemas to exclude from ingestion.                                                                                                                           |
| `schema_pattern.ignoreCase` |          | `True`             | Whether to ignore case sensitivity during pattern matching.                                                                                                                             |
| `view_pattern.allow`        |          |                    | List of regex patterns for views to include in ingestion.                                                                                                                               |
| `view_pattern.deny`         |          |                    | List of regex patterns for views to exclude from ingestion.                                                                                                                             |
| `view_pattern.ignoreCase`   |          | `True`             | Whether to ignore case sensitivity during pattern matching.                                                                                                                             |
| `include_tables`            |          | `True`             | Whether tables should be ingested.                                                                                                                                                      |
| `include_views`             |          | `True`             | Whether views should be ingested.                                                                                                                                                       |
| `include_table_lineage`     |          | `True`             | Whether table lineage should be ingested.                                                                                                                                               |
| `table_lineage_mode`        |          | `"stl_scan_based"` | Which table lineage collector mode to use                                                                                                                                               |
| `include_copy_lineage`      |          | `True`             | Whether lineage should be collected from copy commands                                                                                                                                  |
| `default_schema`            |          | `"public"`         | The default schema to use if the sql parser fails to parse the schema with `sql_based` lineage collector                                                                               |

## Compatibility

Coming soon!

## Lineage

There are multiple lineage collector implementations as Redshift does not support table lineage out of the box.

### stl_scan_based
The stl_scan based collector uses Redshift's [stl_insert](https://docs.aws.amazon.com/redshift/latest/dg/r_STL_INSERT.html) and [stl_scan](https://docs.aws.amazon.com/redshift/latest/dg/r_STL_SCAN.html) system tables to
discover lineage between tables.
Pros:
- Fast
- Reliable

Cons:
- Does not work with Spectrum/external tables because those scans do not show up in stl_scan table.
- If a table is depending on a view then the view won't be listed as dependency. Instead the table will be connected with the view's dependencies.

### sql_based
The sql_based based collector uses Redshift's [stl_insert](https://docs.aws.amazon.com/redshift/latest/dg/r_STL_INSERT.html) to discover all the insert queries
and uses sql parsing to discover the dependecies.

Pros:
- Works with Spectrum tables
- Views are connected properly if a table depends on it

Cons:
- Slow.
- Less reliable as the query parser can fail on certain queries

### mixed
Using both collector above and first applying the sql based and then the stl_scan based one.

Pros:
- Works with Spectrum tables
- Views are connected properly if a table depends on it
- A bit more reliable than the sql_based one only

Cons:
- Slow
- May be incorrect at times as the query parser can fail on certain queries

# Note
- The redshift stl redshift tables which are used for getting data lineage only retain approximately two to five days of log history. This means you cannot extract lineage from queries issued outside that window.

# Redshift-Usage
This plugin extracts usage statistics for datasets in Amazon Redshift. For context on getting started with ingestion, check out our [metadata ingestion guide](../README.md).

Note: Usage information is computed by querying the following system tables - 
1. stl_scan
2. svv_table_info
3. stl_query
4. svl_user_info

##Setup
To install this plugin, run `pip install 'acryl-datahub[redshift-usage]'`.

##Capabilities
This plugin has the below functionalities -
1. For a specific dataset this plugin ingests the following statistics - 
   1. top n queries.
   2. top users.
   3. usage of each column in the dataset.
2. Aggregation of these statistics into buckets, by day or hour granularity.

## Sample usage recipe

Check out the following recipe to get started with ingestion! See [below](#config-details) for full configuration options.

For general pointers on writing and running a recipe, see our [main recipe guide](../README.md#recipes).

```yml
source:
  type: redshift-usage
  config:
    # Coordinates
    host_port: db_host:port
    database: dev
    email_domain: acryl.io

    # Credentials
    username: username
    password: "password"

sink:
# sink configs
```

### Config details
Note that a `.` is used to denote nested fields in the YAML recipe.

By default, we extract usage stats for the last day, with the recommendation that this source is executed every day.

| Field                       | Required | Default                                                        | Description                                                                                                                                                                             |
| --------------------------- | -------- | ---------------------------------------------------------------| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `username`                  |          |                                                                | Redshift username.                                                                                                                                                                      |
| `password`                  |          |                                                                | Redshift password.                                                                                                                                                                      |
| `host_port`                 | ✅       |                                                                | Redshift host URL.                                                                                                                                                                      |
| `database`                  |          |                                                                | Redshift database.                                                                                                                                                                      |
| `env`                       |          | `"PROD"`                                                       | Environment to use in namespace when constructing URNs.                                                                                                                                 |
| `options.<option>`          |          |                                                                | Any options specified here will be passed to SQLAlchemy's `create_engine` as kwargs.<br />See https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create_engine for details. |
| `email_domain`              | ✅       |                                                                | Email domain of your organisation so users can be displayed on UI appropriately.                                                                                                        |
| `start_time`                |          | Last full day in UTC (or hour, depending on `bucket_duration`) | Earliest date of usage to consider.                                                                                                                                                     |   
| `end_time`                  |          | Last full day in UTC (or hour, depending on `bucket_duration`) | Latest date of usage to consider.                                                                                                                                                       |
| `top_n_queries`             |          | `10`                                                           | Number of top queries to save to each table.                                                                                                                                            |
| `bucket_duration`           |          | `"DAY"`                                                        | Size of the time window to aggregate usage stats.                                                                                                                                       |

## Questions

If you've got any questions on configuring this source, feel free to ping us on [our Slack](https://slack.datahubproject.io/)!
