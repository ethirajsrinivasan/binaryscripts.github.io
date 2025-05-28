---
layout: post
title: Building Fault Tolerant Data Pipelines with Pulsar and Apache Hudi
subtitle: Design resilient streaming data pipelines using Apache Pulsar and Apache Hudi with exactly-once semantics and time-travel capabilities
categories: Pulsar
tags: [Pulsar, Apache Hudi, Fault Tolerance, Data Pipelines, Streaming, Lakehouse, Real-Time]
excerpt: Learn how to build fault-tolerant data pipelines using Apache Pulsar and Apache Hudi. This guide covers architecture, ingestion, deduplication, and recovery techniques for robust real-time data lakes.
---
Modern data platforms demand pipelines that are **resilient**, **reliable**, and **real-time**. Failures in ingestion, network, or processing logic shouldn’t compromise the consistency or completeness of data. By combining **Apache Pulsar**, a distributed event streaming platform, with **Apache Hudi**, a transactional data lake framework, you can build **fault-tolerant streaming pipelines** that scale.

In this post, we explore how to use Pulsar and Hudi to design **end-to-end resilient pipelines**, covering ingestion architecture, deduplication, schema handling, and recovery from failures.

---

#### Why Combine Pulsar and Hudi?

| Feature                    | Apache Pulsar                         | Apache Hudi                                |
|----------------------------|----------------------------------------|--------------------------------------------|
| Messaging                  | Distributed publish-subscribe         | —                                          |
| Streaming Ingestion        | Low-latency, high-throughput          | Supports streaming writes via Spark/Flink  |
| Storage                    | Ephemeral                             | Durable lakehouse format (on S3/HDFS)      |
| Fault Tolerance            | Acknowledgments, persistence          | Time-travel, rollback, schema evolution    |
| Deduplication              | At message level (Key_Shared)         | At record level via precombine keys        |

Together, they form a **streaming + storage** pattern with durability and consistency guarantees.

---

#### Pipeline Architecture

```
[Producers / IoT Devices]  
↓  
[Apache Pulsar Topics]  
↓  
[Stream Processing (Flink / Spark Structured Streaming)]  
↓  
[Apache Hudi Tables on S3 / HDFS]  
↓  
[Query Layer: Presto / Trino / Athena / Hive]  
```

---

#### Step 1: Ingesting Data into Pulsar

Produce messages to Pulsar using a schema-encoded format like **Avro** or **JSON**:

```json
{
"device_id": "sensor-001",
"event_time": "2024-11-16T10:00:00Z",
"temperature": 25.3,
"status": "ok"
}
```

Use **persistent topics** with **Key_Shared** or **Shared** subscriptions for parallelism and ordering.

---

#### Step 2: Streaming Into Hudi Using Apache Flink

Pulsar provides Flink connectors that can write directly to Hudi.

Sample Flink job logic:

```java
DataStream<RowData> pulsarStream = env.fromSource(
PulsarSourceBuilder.build("pulsar://broker:6650", "iot-topic"),
WatermarkStrategy.noWatermarks(),
"pulsar-source"
);

Configuration hudiConf = new Configuration();
hudiConf.setString(FlinkOptions.PATH, "s3://hudi/iot-events");
hudiConf.setString(FlinkOptions.TABLE_TYPE, "MERGE_ON_READ");
hudiConf.setString(FlinkOptions.RECORD_KEY_FIELD, "device_id");
hudiConf.setString(FlinkOptions.PRECOMBINE_FIELD, "event_time");

pulsarStream.addSink(HudiSinkProvider.createSink(hudiConf));
```

This configuration ensures **upserts** and **record-level deduplication**.

---

#### Step 3: Ensuring Fault Tolerance

- **Pulsar** ensures message durability via **acknowledgments** and **bookkeeper-based persistence**
- **Flink/Spark** use **checkpointing** and **exactly-once semantics**
- **Hudi** writes are **atomic** — failed writes are rolled back using commit timelines

Use the **precombine field** to keep the latest version of each event:

```
hoodie.datasource.write.precombine.field = event_time
```

This avoids duplicates during retries or replays.

---

#### Step 4: Recovery and Reprocessing

To reprocess or recover data:
- Replay Pulsar messages from **earliest** or **timestamp**
- Use Hudi’s **incremental query** to resume processing:

```python
spark.read.format("hudi")
.option("hoodie.datasource.query.type", "incremental")
.option("hoodie.datasource.read.begin.instanttime", "20241116093000")
.load("s3://hudi/iot-events")
```

- Use Hudi CLI to **rollback failed commits**:

```bash
hudi-cli
> connect --path s3://hudi/iot-events
> rollback 20241116103000
```

---

#### Step 5: Schema Evolution and Compatibility

Enable schema evolution in both Pulsar and Hudi:

- Pulsar:
  ```bash
  bin/pulsar-admin schemas upload --filename event-schema.avsc persistent://tenant/ns/topic
  ```

- Hudi:
  ```
  hoodie.datasource.write.schema.evolution.enable = true
  ```

Always keep fields **nullable** when adding new ones.

---

#### Monitoring and Observability

Monitor your pipeline with:

- **Pulsar metrics**: Broker throughput, consumer lag
- **Hudi timeline**: Commit stats, compaction status
- **Processing framework**: Flink / Spark UI, checkpoints
- **Dashboards**: Prometheus + Grafana, CloudWatch, or Datadog

---

#### Best Practices

- Use **Key_Shared subscription** for deduplicated parallelism
- Use **event_time** as precombine field for ordering
- Enable **Hudi metadata table** for faster file listing
- Use **inline compaction and clustering** for MOR tables
- Monitor **commit latency** and **record size** for tuning

---

#### Conclusion

By combining Apache Pulsar with Apache Hudi, you can build **end-to-end, fault-tolerant data pipelines** that are real-time, consistent, and scalable. Pulsar ensures reliable ingestion, while Hudi guarantees ACID semantics and time-travel — making this duo a powerful foundation for building modern data lakes and analytics platforms.

Embrace this architecture to build pipelines that are resilient by design and ready for scale.
