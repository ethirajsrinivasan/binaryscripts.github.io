---
layout: post
title: Using Hudi with Apache Spark for Real Time Data Lakes
subtitle: Build real-time, ACID-compliant data lakes using Apache Hudi and Apache Spark
categories: Hudi
tags: [Apache Hudi, Spark, Real-Time Data Lakes, Big Data, Lakehouse, Streaming, ACID]
excerpt: Learn how to use Apache Hudi with Apache Spark to create real-time data lakes. Explore features like upserts, incremental queries, and ACID compliance for modern big data architectures.
---
Traditional data lakes based on HDFS or cloud object storage often suffer from **eventual consistency**, **lack of ACID guarantees**, and **high latency**. Modern use cases like **streaming ingestion**, **real-time analytics**, and **data freshness requirements** call for more powerful frameworks.

This is where **Apache Hudi** (Hadoop Upserts Deletes and Incrementals) shines. It brings **streaming primitives**, **transactional guarantees**, and **incremental processing** to data lakes — especially when paired with **Apache Spark**.

In this post, we’ll explore how to use **Hudi with Apache Spark** to build scalable, ACID-compliant **real-time data lakes** for modern data engineering needs.

---

#### What is Apache Hudi?

**Apache Hudi** is an open-source transactional data lake platform that enables:
- **Upserts** and **deletes** on Parquet data
- Near real-time ingestion
- ACID transactions on files stored in HDFS, S3, or GCS
- Incremental data consumption
- Native integration with Spark, Flink, Presto, and Hive

Hudi supports two core table types:
1. **Copy-on-Write (COW)** – writes a new version of the file for every update
2. **Merge-on-Read (MOR)** – maintains a delta log that is compacted periodically

---

#### Why Use Hudi with Apache Spark?

Apache Spark provides the perfect companion for Hudi:
- Enables parallel writes at scale
- Supports structured streaming ingestion
- Native support for DataFrames and SQL
- Seamlessly integrates with Hudi write APIs

Together, Spark and Hudi allow you to build a **real-time data lake** capable of handling streaming ingestion and providing fresh data to BI tools and ML pipelines.

---

#### Setting Up Hudi with Spark

To get started, add the following dependencies in your Spark job:

```bash
--packages org.apache.hudi:hudi-spark3-bundle_2.12:0.14.0
--conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer'
```

Ensure Spark version is compatible with the Hudi bundle (e.g., Spark 3.x).

---

#### Writing to a Hudi Table from Spark

You can write a DataFrame to a Hudi table in Copy-on-Write mode like this:

```scala
val hudiOptions = Map(
"hoodie.table.name" -> "user_events",
"hoodie.datasource.write.recordkey.field" -> "user_id",
"hoodie.datasource.write.partitionpath.field" -> "event_date",
"hoodie.datasource.write.table.name" -> "user_events",
"hoodie.datasource.write.operation" -> "upsert",
"hoodie.datasource.write.precombine.field" -> "event_ts",
"hoodie.datasource.write.hive.style.partitioning" -> "true",
"hoodie.datasource.hive_sync.enable" -> "true",
"hoodie.datasource.hive_sync.database" -> "default",
"hoodie.datasource.hive_sync.table" -> "user_events",
"hoodie.datasource.hive_sync.mode" -> "hms"
)

df.write.format("hudi")
.options(hudiOptions)
.mode("append")
.save("s3://your-bucket/hudi/user_events")
```

---

#### Streaming Ingestion with Spark Structured Streaming

Use Hudi + Spark Structured Streaming to ingest real-time data:

```scala
val streamingDF = spark.readStream
.format("kafka")
.option("subscribe", "events")
.load()
.selectExpr("CAST(value AS STRING)")

streamingDF.writeStream
.format("hudi")
.options(hudiOptions)
.option("checkpointLocation", "/tmp/hudi-checkpoints/")
.outputMode("append")
.start("s3://your-bucket/hudi/user_events")
```

This enables low-latency ingestion with exactly-once semantics and support for upserts.

---

#### Reading Hudi Tables

You can read Hudi tables using Spark DataFrames:

```scala
val hudiDF = spark.read.format("hudi").load("s3://your-bucket/hudi/user_events")
hudiDF.createOrReplaceTempView("user_events")

spark.sql("SELECT * FROM user_events WHERE user_id = 'abc123'")
```

---

#### Incremental Queries with Hudi

One of Hudi’s best features is **incremental pull**, enabling CDC-style querying:

```scala
val incrDF = spark.read.format("hudi")
.option("hoodie.datasource.query.type", "incremental")
.option("hoodie.datasource.read.begin.instanttime", "20240401000000")
.load("s3://your-bucket/hudi/user_events")
```

Use case: Build ETL pipelines that pull only updated rows since the last run.

---

#### Table Types: COW vs MOR

| Feature              | Copy-on-Write (COW)           | Merge-on-Read (MOR)               |
|----------------------|--------------------------------|------------------------------------|
| Write latency        | Higher                         | Lower                              |
| Read performance     | High                           | Moderate (requires merge)          |
| Storage efficiency   | Lower (full rewrites)          | Higher (appends log files)         |
| Use cases            | BI dashboards, reporting       | Streaming ingestion, real-time ML  |

Choose COW for analytical queries and MOR for write-heavy pipelines.

---

#### Hive and Presto Integration

Enable Hive sync with Hudi:

```scala
"hoodie.datasource.hive_sync.enable" -> "true"
"hoodie.datasource.hive_sync.mode" -> "hms"
"hoodie.datasource.hive_sync.database" -> "analytics"
"hoodie.datasource.hive_sync.table" -> "user_events"
```

Presto/Trino can read Hudi tables directly by querying the same Hive Metastore.

---

#### Benefits of Using Hudi for Real-Time Data Lakes

- ACID-compliant transactions on file-based storage
- High-throughput ingestion via Spark Structured Streaming
- Built-in support for **time travel**, **upserts**, and **incremental pulls**
- Easy interoperability with Hive, Presto, Trino, and Flink
- Native support for object storage (S3, GCS) and HDFS

---

#### Conclusion

By integrating **Apache Hudi with Apache Spark**, you unlock the power to build **real-time, highly reliable, and scalable data lakes**. Hudi extends the traditional data lake model with transactional capabilities, making it easier to manage fast-moving data and power next-generation analytics pipelines.

Whether you're dealing with streaming data, change data capture (CDC), or want faster ingestion without sacrificing ACID guarantees, Hudi + Spark offers a powerful, future-ready solution.
