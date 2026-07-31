---
title: "Blog 1"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# AMAZON ATHENA - ANALYZING DATA ON S3 USING SQL WITHOUT CREATING A DATABASE

While learning AWS, I often thought that querying data required installing a database management system like MySQL or PostgreSQL. However, when exploring **Amazon Athena**, I realized there is a simpler way in some cases.

Amazon Athena is a serverless service that allows you to query data stored on Amazon S3 using familiar SQL syntax. This means I don't need to create a server, install a database, or manage infrastructure to analyze data directly. Athena uses the open-source Trino query engine and supports various data formats such as CSV, JSON, Parquet, ORC, and Avro.

## What can Amazon Athena be used for?

After researching, I found that Athena is suitable for quite a few scenarios, such as:

- Analyzing CSV or JSON files stored on S3.
- Querying logs from CloudTrail, VPC Flow Logs, or Application Logs.
- Analyzing data for reporting purposes.
- Supporting the building of a Data Lake in combination with Amazon S3.
- Acting as a data source for Amazon QuickSight to visualize data.

{{% notice tip %}}
What I find interesting is that as long as the data is on S3, you can use SQL to query it without needing to import it into a database first.
{{% /notice %}}

## Querying data with Amazon Athena

To understand it better, I followed a simple example with a CSV file stored on S3.

**Step 1:** Create an S3 Bucket and upload the `students.csv` data file. The content includes columns like:
- id
- name
- major
- gpa

**Step 2:** Access the AWS Console and open Amazon Athena.

On the first use, Athena will ask you to configure an S3 Bucket to store the query results.

**Step 3:** Create a Database.

```sql
CREATE DATABASE university;
```

**Step 4:** Create an external table referencing the file on S3.

```sql
CREATE EXTERNAL TABLE students (
    id INT,
    name STRING,
    major STRING,
    gpa DOUBLE
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION 's3://your-bucket/students/';
```

{{% notice note %}}
Athena only creates metadata; the data remains on S3 and is not copied elsewhere.
{{% /notice %}}

**Step 5:** Execute a query.

```sql
SELECT name, gpa
FROM students
WHERE gpa >= 3.5;
```

{{% notice note %}}
After a few seconds, the results will be displayed directly on the Athena interface.
{{% /notice %}}

## Advantages

After researching, I found that Amazon Athena has quite a few advantages.

First, there is no need to install or manage database servers. This makes starting data analysis much faster.

Second, Athena supports many different data formats. If the data is stored in formats like Parquet or ORC, query performance will also be better.

Additionally, Athena integrates well with many AWS services like AWS Glue Data Catalog, Amazon QuickSight, and Amazon S3, making it suitable for building data analytics systems.

## Some points to note

Besides the advantages above, I also noticed some things to consider. Athena charges based on the amount of data scanned in each query. Therefore, if the data is not optimized or if you use `SELECT *` on very large files, costs can increase.

According to AWS documentation, to reduce costs and improve performance, you should:
- Use columnar formats like Parquet or ORC.
- Partition the data.
- Only query the necessary columns instead of retrieving all the data.

## When to use it?

In my opinion, Amazon Athena is suitable when:

- You want to analyze data stored on Amazon S3.
- You need to quickly inspect CSV or JSON files.
- Analyzing system logs.
- Building a Data Lake without wanting to operate a separate database.
- Combining with QuickSight to create dashboards.

## Conclusion

After exploring, I found that Amazon Athena is a very convenient service for data analysis problems. With just Amazon S3 and a few SQL commands, I was able to query data without deploying an additional database management system. I think this is a service worth trying if you are learning about Data Analytics or working on projects that require data processing on AWS.

## References
1. [AWS Documentation – Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/what-is.html)
2. [Getting Started with Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/getting-started.html)
3. [Amazon Athena User Guide](https://docs.aws.amazon.com/athena/latest/ug/)
4. [Amazon Athena Pricing](https://aws.amazon.com/athena/pricing/)