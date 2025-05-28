---
layout: post
title: Using Pulsar with Apache Hudi for Real Time Data Lake Pipelines
subtitle: Build real-time lakehouse pipelines by integrating Apache Pulsar with Apache Hudi on cloud-native storage
categories: Pulsar
tags: [Pulsar, Hudi, Real-Time, Data Lake, Streaming, Lakehouse, Apache Hudi, Apache Pulsar]
excerpt: Learn how to integrate Apache Pulsar and Apache Hudi to build low-latency, real-time data lake pipelines with ACID guarantees, incremental processing, and unified batch-stream storage.
---
Modern data platforms require **real-time ingestion** and **ACID-compliant storage** to support streaming analytics, machine learning, and reporting. By combining **Apache Pulsar** and **Apache Hudi**, you can build a **high-performance, cloud-native data lake pipeline** that supports both **real-time and batch workloads**.

This blog walks through how to integrate **Pulsar with Hudi**, key architectural considerations, and best practices for building scalable, event-driven lakehouse pipelines.

---

#### Why Pulsar + Hudi?

| Component       | Role                                          |
|-----------------|-----------------------------------------------|
| **Apache Pulsar** | Acts as a scalable, multi-tenant event bus     |
| **Apache Hudi**  | Writes data to data lakes with ACID semantics |

Together they enable:
- **Event ingestion at scale**
- **Real-time updates and deletes**
- **Time-travel queries**
- **Unified batch and streaming workflows**

---

#### Architecture Overview

```
[Data Sources / Events]  
↓  
[Apache Pulsar Topics]  
↓  
[Flink / Spark Structured Streaming]  
↓  
[Apache Hudi Tables on S3 / HDFS]  
↓  
[Query Engines: Presto, Trino, Athena, Spark SQL]  
```

Pulsar ingests the events, while Hudi ensures they are written transactionally to the data lake.

---

#### Setting Up Apache Pulsar

Deploy Pulsar using:
- Kubernetes (Helm charts)
- Binary release
- StreamNative or Pulsar Manager for easier ops

Create a topic for ingestion:

```bash
bin/pulsar-admin topics create persistent://tenant/ns/events-topic
```

Ingest JSON/Avro/Protobuf messages using producers in Java, Python, or Go.

---

#### Reading from Pulsar and Writing to Hudi

Use **Apache Flink** or **Apache Spark Structured Streaming** to connect Pulsar to Hudi.

##### Option 1: Spark Structured Streaming

Set up the Pulsar source:

```python
df = spark \
.readStream \
.format("pulsar") \
.option("service.url", "pulsar://localhost:6650") \
.option("topic", "persistent://tenant/ns/events-topic") \
.load()
```

Parse and write to Hudi:

```python
df_parsed = df.selectExpr("CAST(value AS STRING) as json") \
.select(from_json("json", schema).alias("data")).select("data.*")

hudi_options = {
'hoodie.table.name': 'realtime_events',
'hoodie.datasource.write.recordkey.field': 'event_id',
'hoodie.datasource.write.partitionpath.field': 'event_date',
'hoodie.datasource.write.precombine.field': 'event_ts',
'hoodie.datasource.write.table.type': 'MERGE_ON_READ',
'hoodie.datasource.write.operation': 'upsert',
'hoodie.datasource.hive_sync.enable': 'true',
'hoodie.datasource.hive_sync.mode': 'glue'
}

df_parsed.writeStream \
.format("hudi") \
.options(**hudi_options) \
.outputMode("append") \
.option("checkpointLocation", "s3://checkpoints/hudi/stream/") \
.start("s3://lake/hudi/realtime_events")
```

---

#### Benefits of Using Pulsar with Hudi

- ✅ **Event time processing** using `event_ts` as precombine key
- ✅ **Scalable ingestion** with Pulsar’s topic segmentation
- ✅ **Stream and batch ingestion** into the same Hudi table
- ✅ **Time-travel and incremental views** for downstream engines
- ✅ **Built-in compaction and clustering** for optimized lakehouse layout

---

#### Incremental Querying

Once data is written to Hudi, downstream consumers can read it incrementally:

```python
df = spark.read.format("hudi") \
.option("hoodie.datasource.query.type", "incremental") \
.option("hoodie.datasource.read.begin.instanttime", "20240415120000") \
.load("s3://lake/hudi/realtime_events")
```

---

#### Best Practices

- Use **Avro** or **Protobuf** for schema enforcement
- Enable **schema evolution** in Hudi for changing event structures
- Tune **compaction** frequency based on event volume
- Use **async compaction** and **metadata table** for performance
- Partition data by **date** or **region** to reduce query latency
- Secure Pulsar and S3 using **TLS**, **ACLs**, and **role-based access**

---

#### Monitoring and Scaling

Monitor:
- Pulsar throughput, backlog, topic lag
- Hudi commit frequency and compaction time
- Checkpoint lag in Spark/Flink

Scale:
- Pulsar by adding brokers or Bookies
- Hudi ingestion jobs by adjusting Spark executor counts

---

#### Conclusion

Combining **Apache Pulsar** and **Apache Hudi** creates a powerful foundation for real-time, ACID-compliant data lake pipelines. Pulsar brings scalable event ingestion, while Hudi ensures transactional writes and queryable history.

This integration is ideal for organizations building **lakehouse architectures**, enabling real-time analytics, CDC, and ML pipelines — all backed by open-source technology.
