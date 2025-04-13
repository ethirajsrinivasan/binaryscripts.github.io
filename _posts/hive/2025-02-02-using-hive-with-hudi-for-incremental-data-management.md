---
layout: post
title: Using Hive with Hudi for Incremental Data Management
subtitle: Enable real-time data updates and incremental ETL in Hive using Apache Hudi
categories: Hive
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: [Hive, Hudi, Big Data, Incremental ETL, Data Lake, HDFS, Apache Hudi]
excerpt: Learn how to integrate Hive with Apache Hudi for efficient incremental data processing. Enable real-time data ingestion, ACID compliance, and data versioning in your data lake architecture.
---
Modern data architectures demand the ability to handle **incremental data ingestion**, real-time updates, and low-latency analytics. While Hive excels at querying large datasets in batch mode, it falls short when it comes to updates, deletes, and record-level operations.

Enter **Apache Hudi** — a data lake platform built for **incremental data management** on top of HDFS or cloud object stores. By integrating Hive with Hudi, you can enable **upserts**, **deletes**, **record-level indexing**, and **query optimizations** — without abandoning your existing Hive infrastructure.

This post will guide you through how to use **Hive with Hudi** for efficient and real-time data management at scale.

---

#### What is Apache Hudi?

**Apache Hudi (Hadoop Upserts Deletes and Incrementals)** is a transactional data lake framework that supports:

- Insert, update, and delete operations
- Record-level ACID transactions on HDFS/S3
- Data versioning and rollback
- Incremental data pulls for ETL pipelines
- Real-time querying through Hive, Presto, and Spark

It stores data in **Copy-on-Write (CoW)** or **Merge-on-Read (MoR)** formats, enabling trade-offs between read/write performance.

---

#### Why Use Hudi with Hive?

Traditional Hive tables are immutable — making updates and deletes inefficient and costly. By using Hudi, you get:

- **Efficient upserts and deletes** in Hive-managed tables
- **ACID guarantees** on top of HDFS
- Ability to perform **incremental queries** for real-time pipelines
- Seamless integration with Hive metastore and SQL queries

This allows Hive users to modernize their batch processing with near-real-time capabilities.

---

#### Setting Up Hive with Hudi

To query Hudi tables from Hive:

1. Use Hive 3.1.2+ (or a Hive version compiled with Hudi support)
2. Add the Hudi Hive bundle to your Hive CLI or HiveServer2 classpath:

```bash
export HIVE_AUX_JARS_PATH=/path/to/hudi-hive-bundle.jar
```

3. Use Spark or Hudi DeltaStreamer to write data into Hudi tables.

---

#### Creating a Hudi Table

Create a Hudi table using Spark or the Hudi CLI:

```sql
CREATE TABLE hudi_orders (
order_id STRING,
customer_id STRING,
amount DOUBLE,
order_date STRING
)
USING hudi
OPTIONS (
type = 'cow',
primaryKey = 'order_id',
preCombineField = 'order_date'
)
PARTITIONED BY (order_date);
```

Alternatively, use Hudi DeltaStreamer for ingestion pipelines.

---

#### Querying Hudi Tables from Hive

Once registered with the Hive metastore, Hudi tables can be queried like regular Hive tables:

```sql
SELECT * FROM hudi_orders WHERE order_date = '2024-11-16';
```

For Merge-on-Read (MoR) tables, you can choose between snapshot or read-optimized views:

```sql
-- Snapshot view (latest data with merge)
SELECT * FROM hudi_orders_rt;

-- Read-optimized view (best read performance)
SELECT * FROM hudi_orders_ro;
```

Make sure to set the correct SerDe and input formats when manually creating tables in Hive.

---

#### Performing Incremental Queries

One of Hudi’s most powerful features is **incremental pull** — retrieving only new or updated records since a given commit time.

Set Hive configurations:

```sql
SET hoodie.datasource.query.type=incremental;
SET hoodie.datasource.read.begin.instanttime=20240401000000;

SELECT * FROM hudi_orders WHERE order_date = '2024-11-16';
```

Use commit timestamps (in UTC format) to define the range for incremental processing.

---

#### Compaction and Cleaning

For **Merge-on-Read** tables, Hudi stores data in **log files**, which need to be compacted periodically for query performance.

Schedule compaction via:

```bash
spark-submit ... --class org.apache.hudi.utilities.HoodieCompactor ...
```

Also configure cleaner policies to retain only necessary versions:

```properties
hoodie.cleaner.policy=KEEP_LATEST_COMMITS
hoodie.cleaner.commits.retained=10
```

This ensures storage efficiency and prevents metadata bloat.

---

#### Hive ACID vs Hudi

| Feature                   | Hive ACID (ORC)      | Apache Hudi           |
|--------------------------|----------------------|------------------------|
| Update/Delete Support    | Yes (slow for large data) | Yes (efficient)     |
| Real-time Queries        | No                   | Yes (Snapshot/MoR)    |
| Incremental Ingestion    | No                   | Yes                   |
| Integration with Spark   | Limited              | Native                |
| Object Store Support     | No                   | Yes (S3, GCS)         |

Hudi fills the gap where Hive ACID falls short in performance and scalability.

---

#### Best Practices

- Use **Copy-on-Write** for read-heavy workloads
- Use **Merge-on-Read** for write-heavy or streaming use cases
- Compact MoR tables regularly to avoid read overhead
- Use **preCombineField** to deduplicate records on upserts
- Schedule **cleaning** to remove obsolete files and manage storage
- Tune `hoodie.parquet.max.file.size` and commit configs for performance

---

#### Conclusion

Integrating Hive with Hudi empowers your data lake with **incremental data capabilities**, enabling record-level updates, time travel, and real-time analytics. Hudi brings transactional support and schema evolution to Hive environments without sacrificing Hadoop compatibility.

Whether you’re building ETL pipelines, CDC ingestion flows, or modern data lakes, Hudi and Hive together offer a compelling solution for **big data with real-time agility**.
