---
layout: post
title: Exploring Hudi Table Types COW vs MOR
subtitle: Understand the difference between Hudi's Copy-on-Write and Merge-on-Read tables for efficient data lake management
categories: Hudi
tags: [Apache Hudi, COW, MOR, Data Lake, Big Data, Hadoop, Incremental Processing, Lakehouse]
excerpt: Dive into Apache Hudi's two table types — Copy-on-Write (COW) and Merge-on-Read (MOR). Learn how they differ, when to use each, and how they impact data ingestion, querying, and storage in modern data lakes.
---
**Apache Hudi** (Hadoop Upserts Deletes and Incrementals) has emerged as a powerful transactional data lake framework that supports **streaming and batch processing**. It brings ACID semantics and efficient data management to distributed storage systems like HDFS, Amazon S3, or Azure Data Lake.

One of the key design choices in Hudi is the **table type** — specifically, choosing between **Copy-on-Write (COW)** and **Merge-on-Read (MOR)**. This post explores both table types in depth, highlighting the trade-offs, performance implications, and ideal use cases for each.

---

#### Overview of Hudi Table Types

Apache Hudi supports two table types:

| Table Type      | Storage Behavior                                | Query Performance     | Write Performance     |
|------------------|--------------------------------------------------|------------------------|------------------------|
| Copy-on-Write (COW) | Rewrites data files on every update             | Fast (columnar files)  | Slower (due to rewrites) |
| Merge-on-Read (MOR) | Appends delta logs, compacts later             | Slower (needs merge)   | Faster (write-optimized) |

Choosing the right type depends on your **write/read workload**, **query latency requirements**, and **storage cost considerations**.

---

#### Copy-on-Write (COW)

In COW mode:
- Data is stored in **columnar formats** like **Parquet**
- On every **insert/update**, the affected files are **rewritten**
- Offers **excellent read performance**, especially for OLAP workloads

**Use cases:**
- Batch ETL pipelines
- Read-heavy analytics (e.g., BI tools, dashboards)
- Data sets with fewer updates or slow update frequency

**Example config:**

```json
hoodie.table.type = COPY_ON_WRITE
```

**Pros:**
- Fast read queries (columnar access)
- Simplified storage layout
- Ideal for immutable or append-only data

**Cons:**
- Slower write times
- Resource-intensive during updates

---

#### Merge-on-Read (MOR)

In MOR mode:
- Inserts go to **base files (e.g., Parquet)**, while updates are appended to **delta logs (e.g., Avro)**
- Query engines **merge base files + logs at read time**
- Periodic **compaction** rewrites base files for better performance

**Use cases:**
- Real-time ingestion (e.g., CDC streams)
- Write-heavy workloads
- Low-latency update pipelines

**Example config:**

```json
hoodie.table.type = MERGE_ON_READ
```

**Pros:**
- Faster ingestion and updates
- Supports near real-time processing
- Efficient for frequent mutations (upserts/deletes)

**Cons:**
- Slower reads due to on-the-fly merge
- Requires compaction for long-term performance

---

#### Compaction in MOR Tables

Compaction is the process of **merging delta logs with base files** to:
- Improve query speed
- Reduce number of small files
- Maintain columnar format

Scheduled compaction:

```json
hoodie.compact.inline = true
hoodie.compact.inline.max.delta.commits = 5
```

You can also run compaction manually using Hudi CLI or Spark jobs.

---

#### Query Modes: Snapshot vs. Read-Optimized

Hudi supports two read modes:

1. **Snapshot**: Returns the most recent state (base + log for MOR)
2. **Read-optimized**: Only reads base files (faster, may skip recent updates in MOR)

Choose based on your freshness and latency requirements.

---

#### When to Use COW vs MOR

| Scenario                             | Recommended Table Type |
|--------------------------------------|-------------------------|
| Daily batch ETL and BI queries       | COW                     |
| Real-time ingestion with upserts     | MOR                     |
| Append-only datasets                 | COW                     |
| High write frequency (e.g., IoT)     | MOR                     |
| Fast read latency needed             | COW                     |
| Low update-to-read ratio             | COW                     |

For hybrid use cases, MOR offers more flexibility, while COW simplifies storage and query patterns.

---

#### Integrating with Query Engines

Both COW and MOR tables are accessible via:

- **Apache Hive**
- **Apache Spark SQL**
- **Presto/Trino**
- **Apache Flink (via Hudi connector)**

Enable Hudi catalog or use Hudi Hive Sync Tool to keep schema registered.

Example Spark SQL read:

```scala
spark.read.format("hudi")
.option("hoodie.datasource.query.type", "snapshot")
.load("s3://datalake/hudi/orders_mor/")
```

---

#### Best Practices

- Use **COW** when query performance is the top priority
- Use **MOR** for **streaming pipelines or CDC ingestion**
- Schedule **compaction** jobs in MOR to avoid performance degradation
- Monitor small file growth and manage commit frequency
- Use **partitioning** wisely to optimize both write and read paths

---

#### Conclusion

Choosing between **COW and MOR** in Apache Hudi is crucial for building efficient, scalable, and reliable data lakes. While COW offers simplicity and read speed, MOR brings flexibility and performance for high-velocity, real-time data ingestion.

By understanding the trade-offs and applying them to your data workloads, you can design Hudi tables that meet both your **operational** and **analytical** requirements in the modern **data lakehouse architecture**.
