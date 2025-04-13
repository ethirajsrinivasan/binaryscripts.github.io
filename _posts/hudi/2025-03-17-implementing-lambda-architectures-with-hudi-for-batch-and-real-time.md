---
layout: post
title: Implementing Lambda Architectures with Hudi for Batch and Real Time
subtitle: Use Apache Hudi to unify batch and streaming layers in modern Lambda data architectures
categories: Hudi
tags: [Hudi, Lambda Architecture, Real-Time, Streaming, Batch Processing, Lakehouse, Big Data]
excerpt: Learn how to implement Lambda Architectures using Apache Hudi for unified batch and real-time data processing. Discover best practices for ingestion, compaction, and querying in a lakehouse-style system.
---
The **Lambda Architecture** is a design pattern for building scalable and fault-tolerant big data systems that process data in both **batch** and **real-time** modes. Apache Hudi makes it easier than ever to implement Lambda Architectures by enabling **streaming ingestion**, **batch processing**, and **incremental querying** within a single storage layer.

In this post, we’ll explore how to build a **Lambda Architecture using Apache Hudi**, combining the benefits of real-time responsiveness with the reliability of batch processing — all within a unified data lakehouse.

---

#### What is Lambda Architecture?

Lambda Architecture separates processing into three layers:

1. **Batch Layer** – Computes views from large datasets with full accuracy.
2. **Speed Layer** – Processes real-time data with low latency.
3. **Serving Layer** – Combines both views for querying.

```
[Source Data]
↓
┌───────────────┬────────────────┐
│ Batch Layer    │  Speed Layer   │
│ (e.g. Spark)   │ (e.g. Spark/Flink) │
└────┬───────────┴──────┬────────┘
↓                  ↓
[Apache Hudi Table on S3/HDFS]
↓
[Query Layer: Presto, Athena, Hive]
```

Apache Hudi acts as the **serving layer**, offering **ACID guarantees**, **data versioning**, and **real-time upserts** — which makes it ideal for merging batch and stream data consistently.

---

#### Why Use Apache Hudi for Lambda?

Traditional Lambda implementations often suffer from:

- **Code duplication** between batch and stream pipelines
- **Complex recomputation** for consistency
- **Storage inefficiency**

Hudi helps by:
- **Unifying batch and real-time pipelines**
- **Handling upserts and deletes**
- Providing **incremental views** and **commit timelines**
- Supporting both **Copy-on-Write (COW)** and **Merge-on-Read (MOR)** tables

---

#### Ingesting Batch Data into Hudi

Batch ETL using Spark:

```python
batch_df = spark.read.parquet("s3://raw-data/2024/")

hudi_options = {
'hoodie.table.name': 'events_hudi',
'hoodie.datasource.write.recordkey.field': 'event_id',
'hoodie.datasource.write.precombine.field': 'event_ts',
'hoodie.datasource.write.partitionpath.field': 'event_date',
'hoodie.datasource.write.operation': 'upsert',
'hoodie.datasource.write.table.type': 'COPY_ON_WRITE',
'hoodie.datasource.hive_sync.enable': 'true',
'hoodie.datasource.hive_sync.database': 'default',
'hoodie.datasource.hive_sync.table': 'events_hudi',
'hoodie.datasource.hive_sync.mode': 'glue'
}

batch_df.write.format("hudi") \
.options(**hudi_options) \
.mode("append") \
.save("s3://lake/events_hudi")
```

---

#### Ingesting Real-Time Data into Hudi

Stream data using Spark Structured Streaming:

```python
stream_df = spark.readStream \
.format("kafka") \
.option("subscribe", "events") \
.option("startingOffsets", "earliest") \
.load()

parsed_df = stream_df.selectExpr("CAST(value AS STRING) as json_data") \
.select(from_json("json_data", schema).alias("data")) \
.select("data.*")

parsed_df.writeStream \
.format("hudi") \
.options(**hudi_options) \
.option("checkpointLocation", "s3://checkpoints/hudi/events") \
.outputMode("append") \
.start("s3://lake/events_hudi")
```

Real-time data flows into the **same Hudi table** as batch, with the ability to perform **deduplication** and **incremental merges**.

---

#### Querying with Unified Serving Layer

Apache Hudi supports querying with:

- **Athena**
- **Presto/Trino**
- **Hive**
- **Spark SQL**

Example (Athena):

```sql
SELECT * FROM events_hudi
WHERE event_type = 'login'
AND _hoodie_commit_time > '20240410090000';
```

This allows downstream analytics to **consume fresh data** with transactional guarantees.

---

#### Incremental Processing in Lambda

To process only new data since last run:

```python
begin_time = "20240410090000"

incremental_df = spark.read.format("hudi") \
.option("hoodie.datasource.query.type", "incremental") \
.option("hoodie.datasource.read.begin.instanttime", begin_time) \
.load("s3://lake/events_hudi")
```

Use cases:
- Alerting systems
- Aggregation windows
- Data movement to Redshift, Snowflake, etc.

---

#### Managing Compaction and Clustering

Enable **compaction** for MOR tables:

```
hoodie.compact.inline=true
hoodie.compact.inline.max.delta.commits=10
```

Use **clustering** for better file layout and query performance:

```
hoodie.clustering.inline=true
hoodie.clustering.inline.max.commits=5
```

These ensure efficient performance even as the data grows.

---

#### Best Practices

- Use **precombine keys** for deduplication (e.g., `event_ts`)
- Tune compaction frequency for MOR tables
- Use **partitioning** by date to reduce query scan cost
- Leverage **metadata table** for faster file listing
- Apply **schema evolution** settings to support dynamic fields

---

#### Conclusion

Apache Hudi simplifies the implementation of **Lambda Architectures** by providing a **unified storage layer** that supports both **batch and streaming ingestion** with **real-time querying** capabilities.

With Hudi, you can reduce operational complexity, improve consistency, and scale your lakehouse architecture to meet the demands of modern analytics — all while maintaining **fault-tolerant, low-latency pipelines**.

Embrace Hudi to unify your data lake, power real-time insights, and future-proof your data platform.
