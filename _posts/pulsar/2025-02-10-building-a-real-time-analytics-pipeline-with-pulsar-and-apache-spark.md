---
layout: post
title: Building a Real Time Analytics Pipeline with Pulsar and Apache Spark
subtitle: Harness the power of Apache Pulsar and Spark Structured Streaming to process real-time data at scale
categories: Pulsar
tags: [Pulsar, Apache Spark, Real-Time Analytics, Streaming, Big Data, Event Processing, Data Pipeline]
excerpt: Learn how to design and implement a real-time analytics pipeline using Apache Pulsar and Apache Spark. Explore Pulsar connectors, stream ingestion, transformation, and aggregation for actionable insights.
---
In an era where decisions are increasingly driven by data, businesses need to act on insights in **real time**. Apache Pulsar and Apache Spark together form a robust foundation for **real-time analytics pipelines**. While Pulsar excels in scalable messaging and event delivery, Spark enables complex streaming computations at scale.

This post walks through building a **real-time analytics pipeline** using **Apache Pulsar** as the message broker and **Apache Spark Structured Streaming** for processing — providing a scalable, fault-tolerant framework to derive actionable insights instantly.

---

#### Why Apache Pulsar and Apache Spark?

| Feature                     | Apache Pulsar                          | Apache Spark                          |
|-----------------------------|----------------------------------------|----------------------------------------|
| Message Delivery            | High-throughput pub-sub + queuing      | Native Pulsar integration              |
| Multi-Tenancy & Geo-Rep     | Built-in                               | Compatible with large-scale ingestion  |
| Event Time Support          | Yes                                    | Yes (with watermarking)                |
| Streaming Computation       | No                                     | Yes (Structured Streaming)             |
| Scalability                 | Horizontal (broker + bookies)          | Horizontal (executors + drivers)       |

By combining Pulsar and Spark, you get **real-time messaging** and **distributed stream processing** with rich SQL-like transformations.

---

#### Architecture Overview

```
[Producers] → [Apache Pulsar] → [Spark Structured Streaming] → [Data Lake / Dashboard]
```

- **Producers**: Emit events (e.g., user clicks, IoT metrics)
- **Pulsar**: Buffers and streams events to consumers
- **Spark**: Transforms, aggregates, and persists data
- **Sinks**: Amazon S3, Delta Lake, Elasticsearch, or BI tools

---

#### Step 1: Setting Up Apache Pulsar

Start Pulsar in standalone mode (for local testing):

```bash
bin/pulsar standalone
```

Create a topic:

```bash
bin/pulsar-admin topics create persistent://public/default/user-events
```

Send sample data:

```bash
bin/pulsar-client produce persistent://public/default/user-events \
-m '{"user_id": 101, "action": "click", "ts": 1699999999}'
```

---

#### Step 2: Integrate Spark with Pulsar

Spark supports Pulsar as a source via **pulsar-spark connector**:

Add Maven coordinates:

```xml
<dependency>
<groupId>org.apache.pulsar</groupId>
<artifactId>pulsar-spark-connector_2.12</artifactId>
<version>3.0.0</version>
</dependency>
```

---

#### Step 3: Create Spark Structured Streaming Job

Example Scala code:

```scala
val spark = SparkSession.builder
.appName("Pulsar-Spark Analytics")
.master("local[*]")
.getOrCreate()

val df = spark.readStream
.format("pulsar")
.option("service.url", "pulsar://localhost:6650")
.option("topic", "persistent://public/default/user-events")
.load()

import spark.implicits._
val parsed = df.selectExpr("CAST(value AS STRING) as json")
.select(from_json($"json", schema_of[Event]).as("data"))
.select("data.*")

val agg = parsed
.withWatermark("ts", "2 minutes")
.groupBy(window($"ts", "1 minute"), $"action")
.count()

agg.writeStream
.outputMode("append")
.format("console")
.start()
.awaitTermination()
```

This example:
- Reads from a Pulsar topic
- Parses JSON event messages
- Applies watermarking for late data
- Aggregates by action type and 1-minute windows

---

#### Step 4: Sink to External Systems

You can write the output to:

- **S3 / HDFS** (via Parquet or Delta Lake)
- **Elasticsearch** for real-time dashboards
- **Kafka** or another Pulsar topic for chaining jobs

Example (write to file):

```scala
agg.writeStream
.outputMode("append")
.format("parquet")
.option("path", "/data/analytics/")
.option("checkpointLocation", "/data/checkpoints/")
.start()
```

---

#### Real-World Use Cases

- **E-Commerce**: Analyze user actions for personalization
- **IoT**: Aggregate sensor data across devices in real time
- **Finance**: Monitor fraud patterns using streaming anomaly detection
- **Telecom**: Track usage patterns and latency anomalies live

---

#### Monitoring & Fault Tolerance

- Use **Pulsar Manager UI** to monitor topic throughput and subscriptions
- Configure **Spark Checkpointing** to resume streams after failure
- Use **structured logs** and **metrics sinks** (e.g., Prometheus + Grafana)

---

#### Best Practices

- Use **dedicated Pulsar partitions** for parallel Spark readers
- Set **processingTime triggers** (e.g., every 10s) for predictable latency
- Design **stateless streaming logic** when possible
- Tune **executor memory and cores** for backpressure handling

---

#### Conclusion

Apache Pulsar and Apache Spark form a powerful stack for building **real-time analytics pipelines**. Pulsar’s distributed messaging guarantees and Spark’s streaming engine allow you to build pipelines that are **scalable**, **fault-tolerant**, and **production-ready**.

Whether you're analyzing user activity, monitoring IoT devices, or processing transactions — this combo equips you with the tools to derive insights **as data arrives**.
