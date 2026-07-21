---
title: "Building a Modern Data Lakehouse (Serverless) on AWS"
date: 2026-07-03
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


### PROCESSING BIG DATA AND REDUCING QUERY COSTS BY UP TO 80% WITH AMAZON S3, AWS GLUE, AND AMAZON ATHENA

![Modern Data Lakehouse Architecture](/images/3-BlogsPosted/3.1-Blog1/blog1.png)

When working with Big Data, such as system logs, transaction histories, or user clickstream data, the biggest challenge is not only storing large volumes of data but also querying it efficiently while keeping costs low. Traditional relational databases and data warehouses often become increasingly expensive as data continues to grow.

A modern serverless Data Lakehouse architecture built with **Amazon S3**, **AWS Glue**, and **Amazon Athena** separates storage from compute, providing virtually unlimited scalability while significantly reducing operational costs.

---

### Data Pipeline Architecture

The Data Lakehouse architecture is organized into multiple data zones on Amazon S3 to efficiently manage the data lifecycle:

* **Raw Zone (Amazon S3):** Stores incoming raw data such as CSV files, JSON files, application logs, and other unprocessed datasets.
* **Cataloging (AWS Glue Crawler):** Automatically scans the Raw Zone, detects schemas, creates metadata definitions, and stores them in the AWS Glue Data Catalog.
* **Processing (AWS Glue Job - PySpark):** Performs ETL operations by converting raw data into the Apache Parquet columnar format, partitions the data by year, month, and day, and stores the processed data in the Processed Zone.
* **Analytics (Amazon Athena):** Executes standard SQL queries directly on the processed data stored in Amazon S3 using metadata from the Glue Data Catalog, without requiring a traditional database.

---

### Why Is This Architecture Cost-Effective?

Amazon Athena charges based on the amount of data scanned during query execution (approximately **$5 per TB scanned**). For example, running a simple `SELECT count(*)` query against a 100 GB CSV file requires Athena to scan the entire dataset.

To significantly reduce query costs, two optimization techniques should be applied during the ETL process:

1. **Convert Data to Apache Parquet**

   Apache Parquet stores data in a columnar format. When querying only specific columns, such as `user_id` and `total_amount`, Athena scans only those columns instead of the entire dataset. This optimization can reduce the amount of scanned data by **60% to 80%**.

2. **Data Partitioning**

   Processed data is organized into partitioned folders, for example:

   `s3://processed-zone/year=2026/month=07/day=03/`

   When queries include filters such as:

   `WHERE year = '2026' AND month = '07'`

   Athena scans only the relevant partitions instead of the entire dataset, resulting in faster queries and significantly lower costs.

---

### Hands-on Implementation

* **Step 1 (Storage):** Create two Amazon S3 buckets representing the `raw-zone` and `processed-zone`. Upload a sample dataset such as `sales_data.csv`.

* **Step 2 (Metadata):** Configure an AWS Glue Crawler to scan the `raw-zone`. After execution, the crawler automatically creates a table in the AWS Glue Data Catalog with detected columns and appropriate data types.

* **Step 3 (ETL with PySpark):** Create an AWS Glue Job using either Glue Studio or custom PySpark code. The job reads the raw data, converts it into the Apache Parquet format, partitions the dataset by date, and writes the output to the `processed-zone`.

* **Step 4 (Query):** Open Amazon Athena, select the database from the AWS Glue Data Catalog, and execute SQL queries directly against the processed data.

---

### Conclusion

A serverless Modern Data Lakehouse architecture provides three major advantages:

* **Zero Infrastructure Management** - No servers need to be provisioned or maintained.
* **Automatic Scalability** - Storage and compute resources scale automatically based on workload.
* **Cost Optimization** - Combining Apache Parquet with partitioning significantly reduces query costs.

This architecture also serves as a solid foundation for integrating Business Intelligence tools such as **Amazon QuickSight** or building Machine Learning pipelines using **Amazon SageMaker**.

---

### References

* AWS Lakehouse Architecture Guide
  https://vntechies.dev/.../aws-lakehouse-architecture-guide

* Modern Data Lake House on AWS
  https://www.linkedin.com/.../modern-data-lake-house...

* [Building Modern Data Lakehouse (Serverless) on AWS - Facebook Discussion](https://www.facebook.com/groups/awsstudygroupfcj/posts/2203105570454403)
