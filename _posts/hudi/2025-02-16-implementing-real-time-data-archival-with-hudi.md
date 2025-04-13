---
layout: post
title: Implementing Real Time Data Archival with Hudi
subtitle: Build scalable real-time archival pipelines using Apache Hudi to manage data growth efficiently
categories: Hudi
tags: [Apache Hudi, Real-Time, Data Archival, Big Data, Hadoop, Lakehouse, Incremental Processing]
excerpt: Learn how to implement real-time data archival with Apache Hudi. This guide covers ingestion, compaction, retention policies, and best practices to manage historical data efficiently in data lakes.
---
As organizations generate massive volumes of data daily — especially from event streams, sensors, or user activity — managing this data over time becomes a key challenge. Storing everything indefinitely leads to rising costs, slower queries, and scalability concerns.

**Apache Hudi** provides a powerful framework to **ingest, store, and archive data in real time**, enabling you to retain just the right amount of fresh and historical data, while offloading or deleting obsolete information efficiently.

In this guide, we’ll explore how to build a **real-time data archival solution with Hudi**, including how to leverage **incremental writes**, **retention policies**, and **efficient storage management**.

---

#### Why Use Hudi for Real-Time Archival?

Apache Hudi excels at:
- **Incremental ingestion** of streaming or batch data
- **ACID-compliant upserts and deletes**
- **Efficient compaction and cleaning mechanisms**
- Integration with **Spark**, **Hive**, **Presto**, and **Trino**

These features make it perfect for **real-time data archiving** where you:
- Continuously ingest new records
- Update existing records
- Retain only recent or relevant data
- Compact and clean older data efficiently

---

#### Architecture Overview

```
+---------------------+
| Streaming Source    |
| (Kafka, Flume, etc.)|
+----------+----------+
|
[Spark Structured Streaming]
|
+----------v----------+
| Apache Hudi Table   |
| (MOR or COW)        |
+----------+----------+
|
+----------v----------+
| Retention & Cleaning|
| Compaction / Archive|
+---------------------+
```

You can choose **Copy-on-Write (COW)** for fast queries or **Merge-on-Read (MOR)** for fast ingestion.

---

#### Creating a Hudi Archival Table

Use Spark to define a Hudi table for archival:

```scala
val hudiOptions = Map(
"hoodie.table.name" -> "event_archive",
"hoodie.datasource.write.table.type" -> "MERGE_ON_READ",
"hoodie.datasource.write.recordkey.field" -> "event_id",
"hoodie.datasource.write.partitionpath.field" -> "event_date",
"hoodie.datasource.write.precombine.field" -> "event_ts",
"hoodie.cleaner.policy" -> "KEEP_LATEST_BY_HOURS",
"hoodie.cleaner.commits.retained" -> "10",
"hoodie.keep.min.commits" -> "20",
"hoodie.keep.max.commits" -> "50"
)

dataframe.write
.format("hudi")
.options(hudiOptions)
.mode(SaveMode.Append)
.save("s3://lake/archive/events/")
```

This config sets the table to keep only the **last few hours or versions** of records, removing stale data automatically.

---

#### Real-Time Ingestion with Spark Structured Streaming

Hudi supports streaming writes from Apache Spark:

```scala
val kafkaDF = spark.readStream
.format("kafka")
.option("subscribe", "event_topic")
.load()

val parsedDF = kafkaDF.selectExpr("CAST(value AS STRING)").as[String]
// Transform value to structured format...

parsedDF.writeStream
.format("hudi")
.options(hudiOptions)
.option("checkpointLocation", "/tmp/hudi-checkpoint")
.outputMode("append")
.start("s3://lake/archive/events/")
```

This allows you to archive events **as they happen**.

---

#### Retention and Cleaning Policies

Hudi uses **cleaning** and **archiving** to manage data lifecycle:

- **Cleaner**: Deletes old file versions and log files
- **Archiver**: Moves commit metadata to archive logs to reduce active timeline size

Set retention policy:

```json
hoodie.cleaner.policy = KEEP_LATEST_COMMITS
hoodie.cleaner.commits.retained = 10
hoodie.keep.min.commits = 20
hoodie.keep.max.commits = 50
```

You can also use **KEEP_LATEST_BY_HOURS** or **KEEP_LATEST_BY_VERSIONS** for time-based retention.

---

#### Archival with DeltaStreamer

Apache Hudi's `DeltaStreamer` CLI tool can also manage ingestion + archival:

```bash
hoodie-delta-streamer \
--table-type MERGE_ON_READ \
--source-class org.apache.hudi.utilities.sources.ParquetDFSSource \
--target-base-path hdfs://warehouse/events \
--target-table event_archive \
--transformer-class org.apache.hudi.utilities.transform.IdentityTransformer \
--props file:///tmp/streamer-config.properties
```

This simplifies setting up production-grade archival jobs.

---

#### Compaction and Query Optimization

For MOR tables, schedule **compaction** to merge logs into base files:

```bash
hoodie-cli
> connect --path s3://lake/archive/events/
> schedule compaction
> run compaction
```

You can also set inline compaction in streaming:

```json
hoodie.compact.inline = true
hoodie.compact.inline.max.delta.commits = 3
```

---

#### Querying Archived Data

Use Spark or Presto to query archived Hudi data:

```sql
SELECT * FROM event_archive
WHERE event_date >= '2024-10-01' AND event_type = 'login';
```

If you’re only interested in compacted data (MOR), use `Read Optimized` query mode.

---

#### Best Practices

- Use MOR tables for high-frequency ingestion; COW for read-optimized use cases
- Define partition paths (e.g., `event_date`) to optimize pruning
- Enable regular compaction and cleaning to manage file count
- Monitor archival storage with metrics from the Hudi Timeline Server
- Use `hoodie.keep.max.commits` and cleaner settings to control data growth

---

#### Conclusion

Apache Hudi makes it easy to build **real-time data archival pipelines** that scale with your ingestion rates and retention needs. By leveraging **incremental processing**, **compaction**, and **cleaning**, you can efficiently manage data growth and keep your lakehouse architecture fast, lean, and queryable.

With the right configuration, Hudi can become your go-to solution for **cost-effective, policy-driven data archiving at scale**.
