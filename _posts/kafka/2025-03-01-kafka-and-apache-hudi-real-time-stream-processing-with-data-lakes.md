---
layout: post
title: Kafka and Apache Hudi Real Time Stream Processing with Data Lakes
subtitle: Combine Apache Kafka and Hudi to enable real-time, transactional data lakes with stream ingestion
categories: Kafka
tags: [Kafka, Hudi, Data Lake, Real-Time Streaming, Stream Processing, Lakehouse, Big Data]
excerpt: Learn how to build real-time, transactional data lakes by integrating Apache Kafka with Apache Hudi. This guide covers architecture, ingestion pipelines, and best practices for building a modern lakehouse platform.
---
Modern data platforms need to handle a constant influx of real-time data from diverse sources — while ensuring **data consistency**, **low-latency availability**, and **scalable storage**. Combining **Apache Kafka** and **Apache Hudi** provides a powerful architecture for building **real-time data lakes** that behave like **lakehouses**, supporting both streaming and batch use cases.

In this post, we explore how **Kafka and Hudi** work together to create **real-time ingestion pipelines** for data lakes on platforms like Amazon S3, HDFS, and Azure Data Lake.

---

#### Why Kafka + Hudi?

Apache Kafka is a **durable, distributed event broker**, ideal for capturing real-time data streams.

Apache Hudi is a **transactional data lake framework** that enables:
- **Upserts/deletes**
- **Time travel**
- **Incremental processing**
- **Efficient querying** from storage like S3 or HDFS

Together, they form a **stream-to-lake** pipeline that:
- Ingests data in real-time
- Performs **idempotent** and **atomic** writes
- Enables querying through Spark, Hive, Trino, and Presto

---

#### Architecture Overview

```
[Producers (Apps / Devices / Logs)]
↓
[Apache Kafka Topics]
↓
[Spark Streaming / Flink Consumer]
↓
[Apache Hudi Table on S3 / HDFS]
↓
[Query Engine: Athena / Trino / Hive / Spark SQL]
```

Kafka acts as the **stream source**, and Hudi acts as the **stream sink** — turning event streams into structured, queryable, and versioned tables.

---

#### Writing Kafka Streams to Hudi

Use **Apache Spark Structured Streaming** to consume from Kafka and write to Hudi:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType, StringType, LongType

spark = SparkSession.builder.appName("kafka-to-hudi").getOrCreate()

schema = StructType() \
.add("user_id", StringType()) \
.add("event_type", StringType()) \
.add("event_ts", LongType())

kafka_df = spark.readStream \
.format("kafka") \
.option("kafka.bootstrap.servers", "localhost:9092") \
.option("subscribe", "user-events") \
.load()

parsed_df = kafka_df.selectExpr("CAST(value AS STRING)") \
.select(from_json(col("value"), schema).alias("data")) \
.select("data.*")

hudi_options = {
'hoodie.table.name': 'user_events_hudi',
'hoodie.datasource.write.recordkey.field': 'user_id',
'hoodie.datasource.write.precombine.field': 'event_ts',
'hoodie.datasource.write.partitionpath.field': 'event_type',
'hoodie.datasource.write.table.type': 'MERGE_ON_READ',
'hoodie.datasource.write.operation': 'upsert',
'hoodie.datasource.hive_sync.enable': 'true',
'hoodie.datasource.hive_sync.database': 'default',
'hoodie.datasource.hive_sync.table': 'user_events_hudi',
'hoodie.datasource.hive_sync.mode': 'hms',
'checkpointLocation': 's3://checkpoints/user-events'
}

parsed_df.writeStream \
.format("hudi") \
.options(**hudi_options) \
.outputMode("append") \
.start("s3://lake/user_events_hudi")
```

---

#### Benefits of Using Hudi with Kafka

- ✅ **End-to-End Real-Time Pipelines**
- ✅ **ACID Compliance on Data Lakes**
- ✅ **Efficient File Management via Compaction**
- ✅ **Support for Incremental Queries and CDC**
- ✅ **Schema Evolution and Partitioning Support**

---

#### Querying Kafka-Ingested Hudi Tables

Once ingested, data is immediately queryable from engines like Athena:

```sql
SELECT * FROM default.user_events_hudi
WHERE event_type = 'click'
AND _hoodie_commit_time > '20240416120000';
```

This allows dashboards, ML models, and ETL jobs to act on **fresh, real-time data**.

---

#### Best Practices

- Use **precombine keys** to resolve duplicates from Kafka replays
- Partition on **low-cardinality columns** like `event_type`, `date`
- Enable **inline compaction** for Merge-on-Read tables:
  ```
  hoodie.compact.inline=true
  hoodie.compact.inline.max.delta.commits=5
  ```
- Use **metadata table** for faster file listing:
  ```
  hoodie.metadata.enable=true
  ```
- Enable **schema evolution**:
  ```
  hoodie.datasource.write.schema.evolution.enable=true
  ```

---

#### Use Cases

- **Clickstream Analytics**: Real-time user interaction tracking
- **IoT Ingestion**: Sensor data pipelines into S3
- **CDC Pipelines**: MySQL/Postgres CDC → Kafka → Hudi
- **Operational Reporting**: Event-driven reporting tables with rollback
- **Data Lake Table Audits**: Leverage Hudi time travel and incremental views

---

#### Conclusion

Combining **Apache Kafka and Apache Hudi** enables the creation of real-time, transactional, and query-ready data lakes that serve both operational and analytical workloads.

By integrating these technologies, organizations can achieve **real-time ETL**, **data consistency**, and **lakehouse flexibility** — with minimal operational overhead. If you're building modern data architectures, Kafka + Hudi is a game-changing pattern that delivers speed and reliability at scale.
