### PAXuseBQML

Predicting visitor purchases with BQML model (ML for Business Professionals)

ใช้ BigQuery ML ของ Google (หลักสูตร Machine Learning for Business Professionals Week 4 ของ Google Cloud)

## Setup and Requirements 
# Task 1: Create a dataset
[url=https://postimg.cc/sGcKq9sF][img]https://i.postimg.cc/sGcKq9sF/Bgml.jpg[/img][/url]

# Task 2: Explore the data

1.The data we will use in this lab sits in the bigquery-public-data project, that is available to all. Let's take a look at a sample of this data.

2.Add the query to Query editor box, and click the Run button.
"#standardSQL
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20160801' AND '20170631'
LIMIT 10000;"

3.Let's save this as the training data. Click on the Save View button to save this query as a view. In the popup, type training_data as the Table Name and click Save.


# Task 3: Create a Model

1.Now replace the query with the following to create a model to predict whether a visitor will make a transaction:

"#standardSQL
CREATE OR REPLACE MODEL `bqml_lab.sample_model`
OPTIONS(model_type='logistic_reg') AS
SELECT * from `bqml_lab.training_data`;"

[Optional] Model information & training statistics


# Task 4: Evaluate the Model

1.Query
"#standardSQL
SELECT
  *
FROM
  ml.EVALUATE(MODEL `bqml_lab.sample_model`);"

In this query, you use the ml.EVALUATE function to evaluate the predicted values against the actual data, and it shares some metrics of how the model performed. You should see a table similar to this


# Task 5: Use the Model

1.Query
"#standardSQL
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews,
  fullVisitorId
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170701' AND '20170801';"
  
  There is the additional fullVisitorId column which you will use for predicting transactions by individual user.The WHERE portion reflects the change in time frame (July 1 to August 1 2017).
  
2.Let's save this July data so we can use it in the next 2 steps to make predictions using our model. Click on the Save View button to save this query as a view. In the popup, type july_data as the Table Name.
3.Predict purchases per country
  
  "#standardSQL
SELECT
  country,
  SUM(predicted_label) as total_predicted_purchases
FROM
  ml.PREDICT(MODEL `bqml_lab.sample_model`, (
SELECT * FROM `bqml_lab.july_data`))
GROUP BY country
ORDER BY total_predicted_purchases DESC
LIMIT 10;"

In this query, you're using ml.PREDICT and the BQML portion of the query is wrapped with standard SQL commands. For this lab you're interested in the country and the sum of purchases for each country, so that's why SELECT, GROUP BY and ORDER BY. LIMIT is used to ensure you only get the top 10 results.

4.Predict purchases per user
"#standardSQL
SELECT
  fullVisitorId,
  SUM(predicted_label) as total_predicted_purchases
FROM
  ml.PREDICT(MODEL `bqml_lab.sample_model`, (
SELECT * FROM `bqml_lab.july_data`))
GROUP BY fullVisitorId
ORDER BY total_predicted_purchases DESC
LIMIT 10;"
