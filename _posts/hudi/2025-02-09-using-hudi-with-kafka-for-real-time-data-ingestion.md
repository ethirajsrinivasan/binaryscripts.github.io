---
layout: post
title: Using Hudi with Kafka for Real Time Data Ingestion
subtitle: Build a real-time lakehouse ingestion pipeline with Apache Hudi and Kafka
categories: Hudi
tags: [Hudi, Kafka, Real-Time, Data Ingestion, Big Data, Lakehouse, Apache Hudi, Streaming]
excerpt: Learn how to use Apache Hudi with Apache Kafka to build real-time data ingestion pipelines into your data lake. Explore architecture, setup, and best practices for near-instant data availability.
---
Modern data-driven applications demand **low-latency data availability** for analytics and decision-making. Traditional batch-based data lakes struggle to meet this need. Enter **Apache Hudi** and **Apache Kafka** — a powerful combination for building **real-time ingestion pipelines** into your data lake or lakehouse architecture.

In this post, we explore how to use **Apache Hudi with Kafka** to capture and store streaming data in real time. We'll walk through the architecture, setup options, and best practices for reliable, scalable ingestion.

---

#### Why Use Apache Hudi and Kafka?

| Feature                      | Apache Kafka              | Apache Hudi                  |
|------------------------------|---------------------------|------------------------------|
| Message Queue                | Yes                       | No                           |
| Storage Layer                | No                        | Yes                          |
| Real-Time Ingestion          | Yes (Producer/Consumer)   | Yes (Streaming & Batch Writes) |
| Incremental Queries          | No                        | Yes                          |
| Upserts/Deletes              | No                        | Yes                          |
| Integration with Hive/Spark  | Limited                   | Native                       |

Together, Kafka and Hudi allow:
- Real-time data ingestion
- ACID-compliant writes to data lake
- Near real-time analytical query support

---

#### Architecture Overview

```
[Kafka Producers] → [Kafka Topic] → [Hudi Streamer or Spark Job] → [Hudi Table in HDFS/S3] → [Query Engines: Presto, Hive, Spark]
```

You can use:
- **Hudi DeltaStreamer** for low-code ingestion
- **Custom Spark Streaming jobs** for full flexibility

---

#### Setting Up Apache Hudi with Kafka

##### 1. Kafka Topic with Streaming Data

Use Kafka Connect or your application to produce JSON/Avro/Parquet messages to a Kafka topic:

```bash
kafka-topics.sh --create --topic hudi-events --bootstrap-server localhost:9092 --partitions 3
```

##### 2. Configure DeltaStreamer

DeltaStreamer reads from Kafka and writes to Hudi tables using checkpointed streaming.

Example config:

```bash
hoodie.deltastreamer.source.class=org.apache.hudi.utilities.sources.JsonKafkaSource
hoodie.deltastreamer.source.kafka.topic=hudi-events
hoodie.deltastreamer.source.kafka.bootstrap.servers=localhost:9092
hoodie.deltastreamer.target.base.path=hdfs:///datalake/hudi/events
hoodie.deltastreamer.target.table.name=hudi_events
hoodie.datasource.write.recordkey.field=id
hoodie.datasource.write.precombine.field=ts
hoodie.datasource.write.table.type=MERGE_ON_READ
```

Run DeltaStreamer:

```bash
spark-submit \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
hudi-utilities-bundle.jar \
--props kafka-source.properties \
--continuous
```

---

#### Write Modes: COPY_ON_WRITE vs MERGE_ON_READ

| Mode           | Characteristics                                    |
|----------------|----------------------------------------------------|
| COPY_ON_WRITE  | Fast reads, slower writes, data stored in Parquet |
| MERGE_ON_READ  | Faster writes, supports incremental queries        |

Use **MERGE_ON_READ** for real-time data with upserts. Choose **COPY_ON_WRITE** if you prioritize query speed.

---

#### Querying Hudi Data in Hive/Spark

Register the table in Hive Metastore:

```sql
CREATE EXTERNAL TABLE hudi_events (
id STRING,
event_type STRING,
ts TIMESTAMP
)
STORED AS PARQUET
LOCATION 'hdfs:///datalake/hudi/events';
```

Query in Spark:

```python
df = spark.read.format("hudi").load("hdfs:///datalake/hudi/events")
df.createOrReplaceTempView("events")
spark.sql("SELECT * FROM events WHERE ts > current_timestamp() - interval 10 minutes").show()
```

---

#### Enabling Incremental Queries

Hudi allows incremental reads based on commit time:

```python
incremental_df = spark.read.format("hudi") \
.option("hoodie.datasource.query.type", "incremental") \
.option("hoodie.datasource.read.begin.instanttime", "20240411000000") \
.load("hdfs:///datalake/hudi/events")
```

This allows consuming only new records since last checkpoint — perfect for downstream pipelines.

---

#### Error Handling and Checkpointing

DeltaStreamer and Spark can checkpoint offsets in HDFS:

- Prevents message loss
- Ensures at-least-once delivery
- Supports restart on failure

Always monitor checkpoint logs and store them in a persistent path.

---

#### Best Practices

- Use **JSON or Avro** for Kafka payloads (schema evolution supported)
- Enable **Hive sync** for SQL-based access
- Use **partitioning** on ingestion timestamp or logical key
- Set `hoodie.cleaner.policy` to manage old versions
- Use `MERGE_ON_READ` with compaction for low-latency ingestion

---

#### Conclusion

Apache Hudi and Kafka together form a **real-time ingestion powerhouse**, giving you the flexibility of Kafka with the durability and ACID semantics of Hudi. With the right configuration, you can deliver **real-time analytics**, **incremental ETL**, and **streaming lakehouse architectures** without compromising performance or consistency.

This architecture empowers teams to build **low-latency, reliable pipelines** that deliver insights from fresh data in seconds.
