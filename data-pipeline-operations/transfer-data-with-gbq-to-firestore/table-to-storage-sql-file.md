# Table to Storage: SQL file

To run a Table to Storage data operation, you first need to prepare a SQL query that will extract the data to export.

The SQL file must contain a BigQuery standard SQL query. You can write it directly in the [BigQuery](https://console.cloud.google.com/bigquery) query editor and then save it into a .sql file.

This query will be executed when the data operation is launched, and the result will be stored in the JSON file specified in your configuration.

{% hint style="info" %}
For a GBQ to Firestore data pipeline, you must at least select a **firestore\_path** column
{% endhint %}

## **📋** Global SQL pattern

In order to use the Python script to load the data into Firestore, the SQL that extracts it must follow a specific pattern.

<table><thead><tr><th width="150">Column</th><th>Descriptions</th></tr></thead><tbody><tr><td><p><strong>timestamp</strong></p><p>type: timestamp</p><p>optional</p></td><td>For your different use cases, it can be interesting to have the last calculation date of your dataset to Firestore.<br>This column is optional but recommanded. </td></tr><tr><td><p><strong>firestore_path</strong></p><p>type: string</p><p><strong>mandatory</strong></p></td><td>Variable read by the Python code to build the target path of documents and collections in Firestore. Each category and sub-category must be separated by a pipe "|". <br><br>⚠ Remember to <strong>remove the "|"</strong> that could be in the variables that are used as path names, or it would be interpreted as a category separator!<br><br>⚠ The Firestore path is a succession of collections and documents. You must at all costs end up on a collection of documents. The defined path must therefore contain an even number of categories and sub-categories. See a screenshot of an exemple below.</td></tr><tr><td><p><strong>other variables</strong></p><p>type: string</p><p>optional</p></td><td><p>The other variables are the ones you want to display in your firestore document. </p><p>You can define as many variable as you like, as described in the first example below.</p><p>We'll even see below how to create sub categories with the second example.</p></td></tr></tbody></table>

## :eye\_in\_speech\_bubble: First SQL example

```sql
SELECT
  CURRENT_TIMESTAMP() AS timestamp,
  CONCAT(
    "tailer-activities-runs", 
    "|", REPLACE(account, "|", "_"), -- do not forget to replace any potential pipes!
    "|", REPLACE(configuration_type, "|", "_"),  
    "|", REPLACE(configuration_id, "|", "_"),  
    "|", "freshness", 
    "|", "job_id", 
    "|", REPLACE(job_id, "|", "_"), 
    "|", "next_execution"
  ) AS firestore_path,
-- here starts the "other variables"  
  account,
  configuration_type,
  configuration_id,
  job_id,
  CONCAT("freshness_", job_id) AS collection_groupe_name,
  last_execution_datetime,
  next_execution_datetime,
  frequence,
  status
FROM
  `tailer-demo.dlk_tailer_bda_freshness.metrics`
WHERE
  (1 = 1) -- you could add filters here
```

You will get a BigQuery Result like this:

![BigQuery Result](<../../.gitbook/assets/image (1) (2).png>)

And after loading it into Firestore (see next pages for the next steps), you create collections and documents as specified in the firestore\_path column and get data like this in Firestore:

![Data stored in Firestore](<../../.gitbook/assets/Capture d’écran 2022-05-19 à 10.33.46.png>)

## :eye\_in\_speech\_bubble: Example 2: create a list in the document

You can also create a list in the document using a "data" column, created using the BigQuery ARRAY\_AGG(STRUCT()) functions.

<table><thead><tr><th width="230.95942720763725">Variables</th><th>Descriptions</th></tr></thead><tbody><tr><td><p><strong>data</strong></p><p>type: struct of string or struct of struct of string</p><p>mandatory</p></td><td>Allows you to create a list of sub-elements that correspond N times to the element (for example for a product, you can create a list of sales sorted by date)</td></tr></tbody></table>

The SQL is more complex. Here is an example:

```sql
WITH
  tmp AS (
  SELECT
    "app-data"|| "|" || "000000" || "|" || "product-details" || "|" ||season_code || "|" ||"references"|| "|" ||reference_color_id AS firestore_path,
    CURRENT_TIMESTAMP() AS extraction_timestamp,
    "000000" AS account,
    season_code,
    reference_color_id,
    ARRAY_AGG(STRUCT(
        date,
        discount_value_,
        sales_)
    ORDER BY date ASC
  ) AS data
  FROM
    `dlk_bda_pa_demo.product_metrics_details`
  WHERE
    (1=1) -- you could add filters here
  GROUP BY
    firestore_path,
    extraction_timestamp,
    account,
    season_code,
    reference_color_id)
SELECT
  extraction_timestamp,
  firestore_path,
  account,
  season_code,
  reference_color_id,
  STRUCT(data AS data) AS details
FROM
  tmp
```

The result looks like this in Firestore:

![Data stored in Firestore](<../../.gitbook/assets/Capture d’écran 2022-05-19 à 14.46.42.png>)

