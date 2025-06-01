---
layout: post
title: Using Hudi with Hive for Batch and Incremental Queries
subtitle: Combine Apache Hudi and Hive to run efficient batch and CDC-style incremental queries on your data lake
categories: Hudi
tags: [Apache Hudi, Hive, Big Data, Incremental Queries, Batch Processing, ACID, Data Lake]
excerpt: Learn how to integrate Apache Hudi with Hive to run high-performance batch and incremental queries. Explore table types, Hive sync, and querying Hudi tables using HiveQL.
---
As data pipelines evolve to support both **batch processing** and **real-time analytics**, there's a growing demand for systems that support **incremental queries**, **ACID guarantees**, and **seamless integration with SQL engines** like Hive.

**Apache Hudi** brings these capabilities to traditional data lakes by enabling **streaming ingestion**, **upserts**, and **time-based querying**. Combined with **Apache Hive**, Hudi allows analysts and data engineers to perform **efficient batch and incremental queries** using familiar SQL syntax.

In this blog, we'll explore how to:
- Integrate **Hudi with Hive**
- Run **batch queries** over large datasets
- Perform **incremental reads** using HiveQL
- Understand how Hive handles Hudi's Copy-on-Write (COW) and Merge-on-Read (MOR) tables

---

#### Why Combine Hudi with Hive?

Apache Hive is widely used for data warehousing on Hadoop and object stores. Hudi enhances Hive by adding:

- **ACID support** on HDFS/S3
- **Upserts and deletes**
- **Incremental query capability**
- **Schema evolution**
- **Time travel and versioned querying**

This combination supports use cases like:
- Change data capture (CDC)
- Real-time reporting
- Partitioned batch aggregations

---

#### Table Types: Copy-on-Write vs Merge-on-Read

| Feature                | Copy-on-Write (COW)             | Merge-on-Read (MOR)               |
|------------------------|----------------------------------|------------------------------------|
| Query Engine           | Hive, Presto, Spark              | Hive (limited support), Spark      |
| Query Performance      | Faster for reads                 | Faster for writes                  |
| File Layout            | Rewrites entire file on update  | Appends delta logs and merges      |
| Best Use Case          | Batch queries, reporting         | Streaming ingestion, CDC           |

For Hive users, **COW tables** are recommended due to **better Hive support**.

---

#### Writing a Hudi Table for Hive

Create a COW table and sync it with Hive using Spark:

```scala
val hudiOptions = Map(
"hoodie.table.name" -> "sales_data",
"hoodie.datasource.write.recordkey.field" -> "sale_id",
"hoodie.datasource.write.partitionpath.field" -> "region",
"hoodie.datasource.write.table.name" -> "sales_data",
"hoodie.datasource.write.operation" -> "upsert",
"hoodie.datasource.write.precombine.field" -> "sale_ts",
"hoodie.datasource.write.hive.style.partitioning" -> "true",
"hoodie.datasource.hive_sync.enable" -> "true",
"hoodie.datasource.hive_sync.database" -> "analytics",
"hoodie.datasource.hive_sync.table" -> "sales_data",
"hoodie.datasource.hive_sync.mode" -> "hms"
)

df.write.format("hudi")
.options(hudiOptions)
.mode("append")
.save("s3://data-lake/hudi/sales_data")
```

This creates and registers the table in Hive Metastore for direct querying.

---

#### Querying Hudi Tables in Hive (Batch Mode)

Hive automatically recognizes the Hudi table if it’s synced. You can query it using standard HiveQL:

```sql
SELECT region, SUM(amount) AS total_sales
FROM sales_data
WHERE sale_date = '2024-11-15'
GROUP BY region;
```

This works the same as querying a native Hive ORC/Parquet table — but now with **ACID support and versioned history** behind the scenes.

---

#### Performing Incremental Queries in Hive

Unlike Spark, Hive does **not natively support incremental query mode via SQL**. However, you can simulate incremental processing using **Hudi metadata columns** like `_hoodie_commit_time`:

```sql
SELECT *
FROM sales_data
WHERE _hoodie_commit_time > '20240415120000';
```

Use this in **ETL pipelines** to fetch only new or changed records since the last checkpoint.

You can also **store commit timestamps** in an external metadata table to persist offsets between jobs.

---
Useful columns available in every Hudi table:
- `_hoodie_commit_time` – Commit timestamp
- `_hoodie_record_key` – Unique record identifier
- `_hoodie_partition_path` – Partition value
- `_hoodie_file_name` – Physical file containing the row

Use these for **auditing**, **debugging**, and **incremental logic**.

---

#### Hive Table DDL for External Hudi Tables

If you want to register an existing Hudi table manually in Hive:

```sql
CREATE EXTERNAL TABLE sales_data (
sale_id STRING,
amount DOUBLE,
sale_ts STRING,
region STRING
)
PARTITIONED BY (region STRING)
STORED AS PARQUET
LOCATION 's3://data-lake/hudi/sales_data'
TBLPROPERTIES (
'hoodie.table.name' = 'sales_data',
'hoodie.datasource.write.recordkey.field' = 'sale_id',
'hoodie.datasource.write.precombine.field' = 'sale_ts',
'hoodie.datasource.write.partitionpath.field' = 'region',
'type' = 'cow'
);
```

Ensure partition metadata is refreshed:

```sql
MSCK REPAIR TABLE sales_data;
```

---

#### Best Practices

- Use **COW tables** for Hive-native workflows
- Sync tables to **Hive Metastore** for full Hive compatibility
- Use **_hoodie_commit_time** for Hive-based CDC logic
- Enable **Hive-style partitioning** for cleaner queries
- Prefer **append mode** for ingestion jobs to preserve transaction semantics

---

#### Conclusion

By integrating **Apache Hudi with Hive**, you get the power of **incremental, ACID-compliant data lake tables** with the flexibility of **batch SQL querying**. Whether you're building nightly aggregates or near-real-time reports, Hudi enhances Hive’s capabilities — bridging the gap between traditional batch pipelines and modern real-time architectures.

This hybrid approach makes Hudi + Hive an ideal combination for **evolving data lakes into lakehouses** that support streaming, CDC, and big data analytics at scale.
