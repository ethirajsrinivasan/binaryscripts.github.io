---
layout: post
title: Implementing Change Data Capture with Hudi
subtitle: Capture and manage real-time data changes using Apache Hudi for scalable data lake updates
categories: Hudi
tags: [Hudi, CDC, Data Lake, Big Data, Streaming, Apache Hudi, Delta, Real-Time]
excerpt: Learn how to implement Change Data Capture (CDC) using Apache Hudi to keep your data lake in sync with real-time source systems. Explore write modes, ingestion patterns, and query capabilities.
---
In modern data platforms, **Change Data Capture (CDC)** is critical for synchronizing source systems with data lakes in near real-time. Traditional batch ETL pipelines fall short when capturing incremental updates, inserts, and deletes efficiently.

**Apache Hudi** is an open-source data lake framework that enables **streaming ingestion and CDC** with support for **incremental updates**, **ACID transactions**, and **efficient querying**. In this blog, we’ll explore how to implement CDC using Hudi, understand its architecture, and walk through real-world ingestion patterns and best practices.

---

#### What is Change Data Capture (CDC)?

CDC refers to capturing changes (inserts, updates, deletes) from source systems and applying them to a target system, such as a data lake, in near real-time.

Common CDC sources:
- Relational databases (MySQL, PostgreSQL, Oracle)
- Kafka topics
- Log-based CDC tools (Debezium, Maxwell, GoldenGate)

The goal is to **avoid full reloads** by only ingesting what has changed.

---

#### Why Use Apache Hudi for CDC?

Apache Hudi offers built-in CDC support with features like:

- **Upsert capability**: Efficiently update and insert data
- **ACID compliance** on data lake tables
- **Incremental queries**: Consume only changed records
- **Time travel**: Query historical snapshots
- **Streaming & batch ingestion** compatibility

Hudi supports two write modes:
- **Copy-on-Write (COW)**: Updates rewrite files; optimized for read-heavy workloads
- **Merge-on-Read (MOR)**: Writes updates as logs and merges on read; optimized for write-heavy workloads

---

#### Hudi Table Types for CDC

1. **Copy-on-Write (COW)**:
  - Updates overwrite entire Parquet files
  - Best for scenarios with fewer updates and more reads

2. **Merge-on-Read (MOR)**:
  - Stores updates in log files, merged during query
  - Better for high-frequency CDC pipelines (e.g., Kafka)

---

#### Ingesting CDC Data into Hudi

Let’s assume we receive CDC data from Kafka or a database via a streaming framework like Apache Spark.

Sample CDC record:

```json
{
"id": "1234",
"name": "Alice",
"email": "alice@domain.com",
"op_type": "update",  // insert, update, delete
"ts": "2024-11-16T08:00:00Z"
}
```

We can ingest this using Spark + Hudi:

```scala
val inputDf = spark.readStream.format("kafka")
.option("subscribe", "cdc_topic")
.load()
.selectExpr("CAST(value AS STRING)")

val parsedDf = inputDf
.select(from_json(col("value"), schema).as("data"))
.select("data.*")

parsedDf.writeStream
.format("hudi")
.option("hoodie.table.name", "user_cdc")
.option("hoodie.datasource.write.recordkey.field", "id")
.option("hoodie.datasource.write.precombine.field", "ts")
.option("hoodie.datasource.write.operation", "upsert")
.option("hoodie.datasource.write.table.type", "MERGE_ON_READ")
.option("checkpointLocation", "/hudi/checkpoints/user_cdc")
.option("path", "/hudi/tables/user_cdc")
.start()
```

---

#### Querying Hudi Tables with CDC Semantics

You can run **incremental queries** to fetch only new or updated data:

```sql
SELECT * FROM hudi_user_cdc
WHERE _hoodie_commit_time > '20241116080000';
```

This allows efficient downstream ingestion or sync with other systems.

You can also query **historical snapshots** for debugging or analytics:

```sql
SELECT * FROM hudi_user_cdc
TIMESTAMP AS OF '2024-11-15 12:00:00';
```

---

#### Handling Deletes in Hudi

Hudi supports deletes via the same write path:

```scala
val deleteDf = parsedDf.filter("op_type = 'delete'")
deleteDf.write.format("hudi")
.option("hoodie.datasource.write.operation", "delete")
.option("hoodie.table.name", "user_cdc")
.option("hoodie.datasource.write.recordkey.field", "id")
.mode(SaveMode.Append)
.save("/hudi/tables/user_cdc")
```

This ensures deleted records are reflected correctly in the data lake.

---

#### Best Practices for CDC with Hudi

- Use **Merge-on-Read** for high-frequency updates
- Use **precombine.field** to track latest record versions
- Deduplicate upstream if using unordered event streams
- Store **commit times** for efficient incremental reads
- Enable **Bloom Index** or **Global Index** for faster record lookup
- Compact regularly to merge logs into base files for performance

---

#### Integrating with Hive and Presto

Once data is written to Hudi, it can be queried using Hive, Presto, or Trino:

```sql
CREATE EXTERNAL TABLE user_cdc (
id STRING,
name STRING,
email STRING
)
STORED AS PARQUET
LOCATION 's3a://your-bucket/hudi/tables/user_cdc';
```

Use Hudi’s Hive sync tool to register and update metadata automatically.

---

#### Conclusion

Apache Hudi makes it easy to implement **Change Data Capture (CDC)** on your data lake by enabling real-time ingestion, ACID-compliant upserts, and efficient querying. Whether you're streaming CDC events from Kafka or ingesting from log-based tools, Hudi provides the transactional layer that traditional lakes lack.

By adopting Hudi, you can turn your data lake into a **real-time, CDC-enabled platform** — perfect for building modern analytics pipelines and machine learning workflows.
