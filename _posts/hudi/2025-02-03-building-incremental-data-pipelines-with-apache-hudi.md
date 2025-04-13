---
layout: post
title: Building Incremental Data Pipelines with Apache Hudi
subtitle: Implement efficient and scalable incremental data pipelines on data lakes using Apache Hudi
categories: Hudi
tags: [Apache Hudi, Incremental ETL, Data Pipelines, HDFS, Big Data, Lakehouse, Streaming]
excerpt: Learn how to build efficient incremental data pipelines using Apache Hudi. This guide covers ingestion modes, write operations, incremental queries, and best practices for near real-time data lake updates.
---
Traditional ETL jobs often involve batch processing of entire datasets, leading to unnecessary I/O and resource consumption. In modern data lake architectures, **incremental data pipelines** allow for efficient ingestion and updating of only the changed data.

**Apache Hudi** (Hadoop Upserts Deletes and Incrementals) is a powerful data lake framework that enables **incremental data ingestion**, **record-level updates**, and **streaming ingestion** on top of storage like HDFS, S3, and cloud data lakes.

In this guide, we'll walk through how to design **incremental data pipelines using Apache Hudi**, including table types, ingestion modes, and integration with Hive and Spark.

---

#### Why Apache Hudi?

Apache Hudi adds transactional capabilities and incremental processing support to your data lake, offering:

- **Upserts and Deletes** on immutable file storage
- **Incremental queries** for changed data
- **Efficient merge-on-read architecture**
- **Streaming and batch ingestion support**
- **Hive, Presto, and Trino compatibility**

Hudi helps bridge the gap between traditional data lakes and modern **lakehouse** architectures.

---

#### Core Hudi Table Types

Hudi supports two storage types:

1. **Copy on Write (CoW):**
  - Data is rewritten during every write
  - Ideal for **read-optimized workloads**
  - Lower read latency, higher write cost

2. **Merge on Read (MoR):**
  - Data is appended in delta logs
  - Compaction merges logs into base files
  - Ideal for **write-heavy** or **streaming** scenarios

Choose based on your latency, frequency, and query needs.

---

#### Hudi Ingestion Modes

Hudi supports three ingestion types:

- **Bulk Insert**: For one-time loads of large historical data
- **Insert**: New rows only
- **Upsert**: New rows + updates based on primary key

Upsert example (Spark):

```scala
val hudiOptions = Map(
"hoodie.table.name" -> "user_events",
"hoodie.datasource.write.recordkey.field" -> "event_id",
"hoodie.datasource.write.partitionpath.field" -> "event_date",
"hoodie.datasource.write.precombine.field" -> "updated_at",
"hoodie.datasource.write.operation" -> "upsert",
"hoodie.datasource.write.table.type" -> "MERGE_ON_READ"
)

data.write
.format("hudi")
.options(hudiOptions)
.mode("append")
.save("hdfs://datalake/user_events/")
```

---

#### Performing Incremental Queries

To consume only **new or changed data**, use Hudi's incremental query mode:

```scala
val commits = spark.read.format("hudi")
.load("hdfs://datalake/user_events/")
.select("_hoodie_commit_time")
.distinct()
.orderBy(desc("_hoodie_commit_time"))
.limit(2)

val beginTime = commits.collect().last.getString(0)

val incremental = spark.read.format("hudi")
.option("hoodie.datasource.query.type", "incremental")
.option("hoodie.datasource.read.begin.instanttime", beginTime)
.load("hdfs://datalake/user_events/")
```

This enables downstream jobs to **process only the deltas**, reducing overhead and latency.

---

#### Hive Integration and Sync

You can register Hudi tables with Hive for SQL querying using the **Hive Sync Tool**:

```bash
spark-submit --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
--master yarn \
--conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer' \
--conf 'spark.sql.catalogImplementation=hive' \
--conf 'spark.hadoop.hive.metastore.uris=thrift://localhost:9083' \
...
```

Or use:

```scala
.option("hoodie.datasource.hive_sync.enable", "true")
.option("hoodie.datasource.hive_sync.database", "default")
.option("hoodie.datasource.hive_sync.table", "user_events")
.option("hoodie.datasource.hive_sync.mode", "hms")
```

This enables querying and joining with other Hive-managed datasets.

---

#### Handling Deletes in Hudi

To delete records, pass the `_hoodie_is_deleted` flag during upsert:

```scala
val deleteDF = originalDF.withColumn("_hoodie_is_deleted", lit(true))

deleteDF.write.format("hudi")
.options(hudiOptions)
.mode("append")
.save("hdfs://datalake/user_events/")
```

Deletes will be handled transactionally and respected in downstream reads.

---

#### Best Practices for Incremental Pipelines

- Use **primary keys and precombine fields** to manage late-arriving or duplicate data
- Choose **CoW** for reporting systems, **MoR** for streaming ETL
- Tune **compaction schedules** to manage read performance in MoR
- Use **incremental query mode** for downstream Spark jobs
- Enable **Hive Sync** for SQL-based consumption and BI tools
- Periodically **clean up old commits** with Hudi's cleaner service

---

#### Monitoring and Debugging

Monitor Hudi pipelines by:
- Reviewing commit timelines in `.hoodie` directory
- Using the Hudi CLI:  
  ```hudi-cli.sh --path hdfs://datalake/user_events/```
- Checking Hive metastore sync logs
- Tracking lag in streaming ingestion jobs

---

#### Conclusion

Apache Hudi brings **incremental processing**, **record-level updates**, and **streaming efficiency** to data lakes. By adopting Hudi, organizations can modernize their ETL pipelines, reduce storage and compute overhead, and build real-time analytics platforms on top of HDFS, S3, or cloud-native object storage.

Whether you're building a lakehouse, running CDC pipelines, or just optimizing batch ETL, **Apache Hudi enables scalable and fault-tolerant incremental data pipelines** for the future of big data.
