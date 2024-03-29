---
description: >-
  Learn how to create the SQL and DDL files corresponding to the workflow tasks
  of a Table to Table data operation.
---

# Table to Table SQL and DDL files

## :map: Overview

A SQL workflow is a sequence of tasks that feed tables in parallel or sequentially.

A workflow can be composed of the following task types:

* SQL task ("sql"): instructions to load, merge and reorganize data.
* Table creation task ("create\_gbq\_table"): instructions to create a destination table.
* Table copy task ("copy\_gbq\_table"): instructions to duplicate a table (provided in the [data operation configuration file](tables-to-tables-configuration-file.md)).

## :oil: SQL tasks

SQL tasks are steps of the workflow. Each SQL task is defined with a .sql file that contains the query. You can write the queries directly in the query editor of [BigQuery](https://console.cloud.google.com/bigquery) and then save them to .sql files.

{% hint style="info" %}
The file can contain a SQL query or a SQL script (like assertions or expectations).
{% endhint %}

{% hint style="info" %}
The name of the SQL file should be the same as the SQL task.
{% endhint %}

Example:

```sql
SELECT
    customer_id,
    optin,
    cast(creation_date as date) as creation_date
FROM `referential.customers`
QUALIFY ROW_NUMBER() OVER(dedup_pk) = 1
WINDOW dedup_pk as (
  PARTITION BY customer_id
  ORDER BY update_date desc, import_date desc, tlr_ingestion_timestamp_utc desc
)
```

## :new: Table creation tasks

Once the SQL queries are ready, you need to use one or several DDL files to create the destination BigQuery tables that will contain the output data.

### DDL Example

```json
{
    "bq_table_description": "Describe the content of the table. The description will be attached to the table bigquery Metadata",
    "bq_table_schema": [
        {
            "name": "field_date",
            "type": "DATE",
            "description": "Describe the field_date"
        },
        {
            "name": "field_2",
            "type": "STRING",
            "description": "Describe the field_2"
        },
        {
            "name": "field_3",
            "type": "INTEGER",
            "description": "Describe the field_3",
            "mode": "REQUIRED"
        }
    ],
    "bq_table_clustering_fields": ["field_2", "field_3"]
    "bq_table_timepartitioning_field": "field_date",
    "bq_table_timepartitioning_type": "MONTH",
    "bq_table_timepartitioning_expiration_ms": "86400000",
    "bq_table_timepartitioning_require_partition_filter": false
}
```

### **DDL Parameters**

| Parameter                                                                                                     | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p><strong>bq_table_description</strong></p><p>type: string</p><p>mandatory</p>                               | Description of the BigQuery table.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| <p><strong>bq_table_schema</strong></p><p>type: array</p><p>mandatory</p>                                     | <p>BigQuery table schema. It contains a list of fields corresponding to the number of columns it will contain.</p><p>Each field described has three attributes:</p><ul><li><strong>name</strong></li><li><a href="table-to-table-sql-and-ddl-files.md#data-types"><strong>type</strong></a></li><li><strong>description</strong></li></ul>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| <p><strong>bq_table_clustering_fields</strong></p><p>type: array</p><p>optional</p>                           | <p>List of fields used when clustering is enabled.</p><p>The table data will be automatically organized based on the contents of the fields you specify. Their order determines the sort order of the data.</p><p>If this parameter is set, time partitioning will be automatically enabled on the table. If you don't set partitioning parameters, default values will be used.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| <p><strong>bq_table_timepartitioning_field</strong></p><p>type: string</p><p>optional</p>                     | <p>If this parameter is set, the table will be partitioned by this field.</p><p>If not, the table will be partitioned by pseudo column <strong>_PARTITIONTIME</strong>.</p><p>The field must be a top-level <strong>TIMESTAMP</strong> or <strong>DATE</strong> field. Its mode must be <strong>NULLABLE</strong> or <strong>REQUIRED</strong>.</p><p>(Refer to <a href="https://googleapis.dev/python/bigquery/latest/generated/google.cloud.bigquery.table.TimePartitioning.html#google.cloud.bigquery.table.TimePartitioning">BigQuery documentation</a> for more information.)</p><p>Note: You can set this parameter to a field that equals to <strong>DATE('')</strong>. Then, if you relaunch an execution with a partition, and if <strong>default_write_disposition</strong> is set to "WRITE_APPEND" in the JSON configuration file, Tailer will check if the corresponding partition already exists in the table:</p><ul><li>If it does, it will delete it, and replace it with the current execution data.</li><li>If not, it will add them.</li></ul><p></p> |
| <p><strong>bq_table_timepartitioning_type</strong><br>type: string</p><p>optional</p>                         | <p>Sets the partition type. Use one of the following values: "HOUR", "DAY", "MONTH" or "YEAR".</p><p>If not present, default is "DAY".</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| <p><strong>bq_table_timepartitioning_expiration_ms</strong></p><p>type: integer</p><p>optional</p>            | <p>Number of milliseconds for which to keep the storage for a partition.</p><p>(Refer to <a href="https://googleapis.dev/python/bigquery/latest/generated/google.cloud.bigquery.table.TimePartitioning.html#google.cloud.bigquery.table.TimePartitioning">BigQuery documentation</a> for more information.)</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| <p><strong>bq_table_timepartitioning_require_partition_filter</strong></p><p>type: boolean</p><p>optional</p> | <p>If set to true, queries over the partitioned table require a partition filter that can be used for partition elimination to be specified.</p><p>(Refer to <a href="https://googleapis.dev/python/bigquery/latest/generated/google.cloud.bigquery.table.Table.html#google.cloud.bigquery.table.Table.require_partition_filter">BigQuery documentation</a> for more information.)</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

### DDL Data Types

Tailer Platform supports the following data types.

#### Numeric types

| Name      | Description                                                                                                                                                                                                                                                                                                    |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `int64`   | <p>Integers are numeric values that do not have fractional components.</p><p>They range from -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807.</p>                                                                                                                                                      |
| `float64` | Floating point values are approximate numeric values with fractional components.                                                                                                                                                                                                                               |
| `numeric` | <p>This data type represents decimal values with 38 decimal digits of precision and 9 decimal digits of scale. (Precision is the number of digits that the number contains. Scale is how many of these digits appear after the decimal point.)</p><p>It is particularly useful for financial calculations.</p> |

#### Boolean type

| Name      | Description                                                                                                                                                      |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `boolean` | This data type supports the `true`, `false`, and `null` values. It can perform some basic conversions, such as `'true'`, `'True'`, `True`, or 1 becoming `true`. |

#### String type

| Name     | Description                                                                                                                                                                                 |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `string` | <p>Variable-length character data.</p><p>When converting data from string to a different data type, makes sure to use <code>safe_cast</code> when you're unsure about the data quality.</p> |

#### Bytes type

| Name    | Description                                                                                                        |
| ------- | ------------------------------------------------------------------------------------------------------------------ |
| `bytes` | Variable-length binary data. This data type is rarely used but can be useful for characters with unusual encoding. |

#### Time types

{% hint style="info" %}
Only the `date`, `datetime` and `timestamp` data types (not `time`) allow table partitioning.
{% endhint %}

{% hint style="info" %}
Time zone management being difficult with BigQuery, prefer the UTC format.
{% endhint %}

| Name        | Description                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `date`      | This data type represents a calendar date. It includes the year, month, and day.                                                                                     |
| `time`      | This data type represents a time, as might be displayed on a watch, independent of a specific date. It includes the hour, minute, second, and subsecond.             |
| `datetime`  | This data type represents a date and time, as they might be displayed on a calendar or clock. It includes the year, month, day, hour, minute, second, and subsecond. |
| `timestamp` | This data type represents an absolute point in time, with microsecond precision.                                                                                     |

## :aquarius: Table copy tasks

If necessary, you can duplicate an existing table using a table copy task, for example to share the contents of a table with a partner inside their own dataset. Although it would be possible to use an SQL script with **SELECT \***, a copy task is more efficient. The parameters set in the JSON configuration file are sufficient, no specific file is required for this type of task.
