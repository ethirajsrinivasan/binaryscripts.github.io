---
layout: post
title: Understanding Hudi Architecture and Core Components
subtitle: Dive into Apache Hudi’s architecture and explore how it enables fast, incremental data ingestion and real-time analytics
categories: Hudi
tags: [Hudi, Apache Hudi, Big Data, Data Lake, Streaming, Incremental Processing, Data Engineering]
excerpt: Learn how Apache Hudi’s architecture works, including core components like Write Client, Timeline Server, DeltaStreamer, and the metadata table. Understand how Hudi supports upserts, time travel, and efficient queries.
---
Modern data lakes need to support **real-time ingestion**, **incremental processing**, and **efficient querying** — all while scaling to handle petabytes of data. **Apache Hudi (Hadoop Upserts Deletes and Incrementals)** addresses these challenges by bringing **transactional capabilities** and **streaming semantics** to data lakes on Hadoop-compatible storage.

In this blog, we’ll explore the **architecture and core components of Apache Hudi**, how it enables ACID-like guarantees on object stores, and the moving parts behind its fast, flexible data ingestion framework.

---

#### What is Apache Hudi?

Apache Hudi is an open-source **data lake platform** that enables:

- **Upserts and deletes** in data lakes
- **Incremental pull** of data changes
- **Time travel and data versioning**
- **Streaming + batch ingestion**
- **Efficient querying** via Hive, Spark, Presto, and Trino

It supports both **copy-on-write (COW)** and **merge-on-read (MOR)** storage strategies, optimized for read-heavy or write-heavy use cases, respectively.

---

#### High-Level Hudi Architecture

At a high level, Hudi consists of the following layers:

1. **Ingestion Layer** (e.g., DeltaStreamer, Spark Jobs)
2. **Storage Layer** (COW/MOR files on HDFS, S3, GCS)
3. **Metadata Layer** (commit timeline, metadata table)
4. **Query Layer** (Hive, Spark SQL, Presto, Trino, Flink)

These layers work together to bring **transactional data management** to large-scale object storage systems.

---

#### Core Components of Hudi

Let’s dive into the major components that make Hudi work:

---

##### 1. Write Client

The **Hudi Write Client** is responsible for ingesting and writing data into Hudi tables. It supports operations like:

- **UPSERT**: Insert new records and update existing ones
- **INSERT**: Insert-only workload
- **BULK_INSERT**: High-throughput batch ingestion
- **DELETE**: Remove records by key

```scala
val hudiOptions = Map(
"hoodie.table.name" -> "user_events",
"hoodie.datasource.write.recordkey.field" -> "user_id",
"hoodie.datasource.write.operation" -> "upsert"
)

data.write
.format("hudi")
.options(hudiOptions)
.mode(SaveMode.Append)
.save("/data/hudi/user_events")
```

This client coordinates with the **Timeline Server** and **metadata table** to manage writes and commit operations.

---

##### 2. Timeline and Commit Protocol

Hudi tracks every operation using a **commit timeline**. Each write generates an **instant** (timestamped action) — like:

- **.commit**: Successfully completed commit
- **.inflight**: In-progress operation
- **.requested**: Operation request logged

These are visible in `.hoodie` metadata directory:

```
/data/hudi/.hoodie/
├── 20240410091523.commit
├── 20240410092000.inflight
└── 20240410093000.requested
```

The timeline enables **rollback**, **clustering**, and **incremental reads**.

---

##### 3. Metadata Table

The **Hudi Metadata Table** stores auxiliary information like:

- File listings
- Column statistics
- Record locations

It improves query planning speed by **eliminating costly file listing operations** — especially helpful on object stores like S3 or GCS.

Enable it via:

```
hoodie.metadata.enable=true
```

---

##### 4. DeltaStreamer

**DeltaStreamer** is a built-in tool for ingesting data from sources like Kafka, Hive, RDBMS (via Sqoop), or files. It supports:

- Continuous or batch ingestion
- Source-to-target schema evolution
- ETL with transformation logic

Run it with a simple config:

```bash
hoodie-delta-streamer \
--table-type COPY_ON_WRITE \
--source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
--target-base-path s3://data/hudi/events \
--target-table events
```

DeltaStreamer helps avoid writing custom Spark jobs for ingestion.

---

##### 5. Storage Types: Copy-on-Write (COW) vs Merge-on-Read (MOR)

| Feature             | Copy-on-Write (COW)        | Merge-on-Read (MOR)         |
|---------------------|----------------------------|------------------------------|
| Write Cost          | Higher (overwrite files)   | Lower (write delta logs)     |
| Read Performance    | Fast                       | Slower (needs merge)         |
| Use Case            | Read-heavy workloads       | Write-heavy + streaming      |
| Compaction Required | No                         | Yes                          |

Choose **COW** for frequent reads and **MOR** for frequent updates or streaming ingestion.

---

##### 6. Query Engines and Access Methods

Hudi integrates with multiple engines:

- **Apache Hive** (via Hive InputFormat)
- **Spark SQL** (`spark.read.format("hudi")`)
- **Presto/Trino** (`hive catalog` with Hudi tables)
- **Flink** for real-time processing

It supports **snapshot queries**, **incremental queries**, and **point-in-time queries**:

**Incremental read example:**

```scala
val df = spark.read.format("hudi")
.option("hoodie.datasource.query.type", "incremental")
.option("hoodie.datasource.read.begin.instanttime", "20240401000000")
.load("/data/hudi/user_events")
```

---

#### Bonus: Key Hudi Table Types

1. **Copy-on-Write Table**: Stores only base files
2. **Merge-on-Read Table**: Stores base files + delta logs
3. **Bootstrap Table**: Helps migrate existing data into Hudi format with minimal rewrite

---

#### Best Practices

- Enable **metadata table** for large datasets
- Use **clustering** to optimize file sizes for downstream engines
- Schedule **compaction** periodically for MOR tables
- Choose **UPSERT vs BULK_INSERT** depending on ingestion frequency
- Monitor commits via timeline for observability and rollback

---

#### Conclusion

Apache Hudi revolutionizes the way we build modern data lakes by supporting **real-time ingestion**, **ACID operations**, and **incremental analytics** at scale. Understanding its architecture — from the Write Client and Timeline to the Metadata Table and DeltaStreamer — equips you to build faster, more reliable, and cost-efficient big data pipelines.

Whether you're ingesting millions of records per hour or supporting time-travel analytics, Hudi offers the flexibility and power to keep your lake fresh and query-ready.
