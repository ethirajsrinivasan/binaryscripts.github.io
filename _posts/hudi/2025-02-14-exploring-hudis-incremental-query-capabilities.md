---
layout: post
title: Exploring Hudi Incremental Query Capabilities
subtitle: Efficiently process and query only changed data using Hudi’s incremental query features
categories: Hudi
tags: [Apache Hudi, Incremental Queries, Big Data, Data Lake, Change Data Capture, Stream Processing]
excerpt: Learn how Apache Hudi enables incremental queries to power near real-time analytics. Discover how to configure and use Hudi's change data capture capabilities for efficient data lake querying.
---
In modern data architectures, it’s inefficient to scan entire datasets for changes. Enterprises need ways to process only the **new or updated data** — also known as **incremental data** — to power real-time analytics, streaming pipelines, and CDC (Change Data Capture) systems.

**Apache Hudi** solves this problem elegantly with its **incremental query capabilities**, allowing you to **read only the changes** made since a specific point in time.

In this guide, we explore how to perform **incremental queries in Apache Hudi**, understand their benefits, and apply them in production use cases like micro-batch ETL, streaming, and data lake synchronization.

---

#### What Are Incremental Queries in Hudi?

Incremental queries in Hudi allow users to fetch records that have changed (inserted, updated, or deleted) **since a specific commit or timestamp**. This is especially useful when:
- Processing CDC events
- Building streaming data pipelines
- Creating materialized views or serving layers

Unlike full table scans, incremental queries return **only new changes**, which makes them ideal for **low-latency and high-efficiency** data ingestion or transformation.

---

#### Hudi Table Types Recap

To use incremental queries, your table must be in one of the supported formats:

1. **Copy-on-Write (CoW)**:
  - Efficient for read-heavy workloads
  - Every update rewrites the entire file

2. **Merge-on-Read (MoR)**:
  - Stores updates as logs
  - Requires compaction for base file optimization

Incremental queries work on both types but are especially powerful with MoR for high-throughput scenarios.

---

#### How Incremental Queries Work

Internally, Hudi tracks all changes using **commit metadata**. Every commit (write, update, delete) gets a unique timestamp (`instant time`). You can use this timestamp to fetch only changes since that point.

Supported operations:
- **Read Inserts/Updates**
- **Read Deletes (soft deletes)**
- **Filter by partition**

---

#### Performing an Incremental Query (Spark SQL)

Here’s how you can perform an incremental query using Apache Spark:

```scala
val beginInstantTime = "20240410010101"

val incrementals = spark.read.format("hudi")
.option("hoodie.datasource.query.type", "incremental")
.option("hoodie.datasource.read.begin.instanttime", beginInstantTime)
.load("s3://datalake/hudi/transactions")

incrementals.show()
```

To get a list of recent commits:

```bash
hdfs dfs -ls /path/to/hudi/.hoodie/
# or use Hudi CLI:
hudi-cli
connect --path /path/to/table
commits show
```

Use the latest timestamp from this list for `begin.instanttime`.

---

#### Filtering Incremental Queries by Partition

To narrow down results to specific partitions:

```scala
.option("hoodie.datasource.read.incr.paths", "/hudi/orders/region=US")
```

This avoids scanning unnecessary partitions and improves performance for geo-segmented or time-partitioned data.

---

#### Use Case 1: Streaming ETL Pipelines

You can poll changes every 10 minutes and write only the new data to a target sink (e.g., Redshift, Cassandra, or Hive):

```scala
val lastCheckpoint = "20240410010101"

val updates = spark.read.format("hudi")
.option("hoodie.datasource.query.type", "incremental")
.option("hoodie.datasource.read.begin.instanttime", lastCheckpoint)
.load("/hudi/orders")

updates.write.format("parquet").save("/staging/orders_delta/")
```

Schedule this as a micro-batch job using Apache Airflow or a Spark Streaming application.

---

#### Use Case 2: Materialized Views and Reporting Layers

Instead of reprocessing the full dataset, generate deltas and apply them to existing aggregations or lookup tables.

```scala
val newUsers = spark.read.format("hudi")
.option("hoodie.datasource.query.type", "incremental")
.option("hoodie.datasource.read.begin.instanttime", "20240411000000")
.load("/hudi/users")

newUsers.groupBy("signup_channel").count().show()
```

This enables **fast, near-real-time dashboards** and **cost-efficient reporting**.

---

#### Use Case 3: Change Data Capture (CDC)

Track row-level changes to replicate Hudi updates downstream into external systems like Kafka, ElasticSearch, or PostgreSQL.

Hudi includes metadata fields:
- `_hoodie_commit_time`
- `_hoodie_record_key`
- `_hoodie_partition_path`

You can use these fields to:
- Identify the operation time
- Generate CDC payloads
- Detect and deduplicate changes

---

#### Limitations and Considerations

- **Commits are immutable**, so changes made outside Hudi won’t show up
- **Incremental queries don't support time ranges** natively — only start time
- Deletes are soft by default — filtering out deleted records may require additional handling
- For full sync jobs, fallback to **snapshot queries**

---

#### Best Practices

- Store commit timestamps in a **checkpoint table** for fault tolerance
- Use **merge-on-read** for high-velocity sources and updates
- Compact regularly to optimize performance for downstream incremental reads
- Avoid backfilling via incremental queries; use snapshot queries for full loads
- Use **partition filters** in large datasets to reduce load

---

#### Conclusion

Apache Hudi’s **incremental query capabilities** offer a powerful way to build **efficient, real-time data pipelines** without resorting to full scans. Whether you're building streaming ETL, change data capture, or delta-aware reporting systems, these features allow you to track, process, and serve fresh data with minimal latency and maximum scalability.

Mastering incremental queries is key to unlocking the full potential of your **data lakehouse architecture**.
