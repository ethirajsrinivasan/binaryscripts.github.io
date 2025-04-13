---
layout: post
title: Using Hudi DeltaStreamer with Kafka for Low Latency Pipelines
subtitle: Build near real-time lakehouse ingestion using Hudi DeltaStreamer and Kafka
categories: Hudi
tags: [Hudi, Kafka, DeltaStreamer, Real-Time, Stream Ingestion, Lakehouse, Big Data]
excerpt: Learn how to set up Hudi DeltaStreamer to ingest data from Apache Kafka into a lakehouse storage layer like S3 or HDFS. Explore configurations, source formats, checkpointing, and tuning for low-latency pipelines.
---
As modern applications generate massive streams of data in real time, engineering teams are challenged with ingesting this data efficiently into data lakes for downstream analytics and machine learning.

**Apache Hudi’s DeltaStreamer**, combined with **Apache Kafka**, offers a powerful solution to build **low-latency pipelines** that continuously ingest and upsert streaming data into a **transactional data lake** — all without writing custom Spark code.

In this post, we’ll explore how to configure **Hudi DeltaStreamer to consume from Kafka**, write to S3 or HDFS, and tune it for scalable, real-time ingestion.

---

#### What is Hudi DeltaStreamer?

**DeltaStreamer** is a Spark-based utility provided by Hudi that supports:

- **Source-to-lake ingestion** from Kafka, files, JDBC, and Hive
- **Upserts, inserts, and deletes** using Hudi table formats
- **Schema evolution**, transformations, and checkpointing
- Both **batch and continuous modes**

It removes the need to write and maintain custom ingestion code for structured streaming jobs.

---

#### Use Case: Kafka to Lakehouse with DeltaStreamer

Architecture overview:

```
Kafka → DeltaStreamer (Spark Job) → Hudi Table (on S3/HDFS) → Athena / Presto / Hive
```

DeltaStreamer reads JSON/Avro events from a Kafka topic and writes them to a Hudi Copy-on-Write (COW) or Merge-on-Read (MOR) table in a transactional and incremental manner.

---

#### Prerequisites

- Apache Hudi 0.10.0 or later
- Apache Kafka running with topics populated
- Apache Spark 3.x cluster (YARN, EMR, or standalone)
- Access to S3 or HDFS
- Schema registry (optional for Avro)

---

#### Step 1: Define Properties File

Create a config file: `kafka-source.properties`

```properties
# Kafka source configs
hoodie.deltastreamer.source.class=org.apache.hudi.utilities.sources.KafkaSource
hoodie.deltastreamer.source.kafka.topic=kafka_orders
hoodie.deltastreamer.source.kafka.bootstrap.servers=broker1:9092,broker2:9092
hoodie.deltastreamer.source.kafka.group.id=hudi_ingest_group
hoodie.deltastreamer.source.kafka.fetch.max.records=1000

# Schema provider
hoodie.deltastreamer.schema.provider.class=org.apache.hudi.utilities.schema.FilebasedSchemaProvider
hoodie.deltastreamer.source.schema.file=/path/to/schema.avsc

# Target Hudi table configs
hoodie.table.name=hudi_orders
hoodie.datasource.write.recordkey.field=order_id
hoodie.datasource.write.precombine.field=event_ts
hoodie.datasource.write.partitionpath.field=region
hoodie.datasource.write.table.type=COPY_ON_WRITE
hoodie.datasource.write.operation=upsert
hoodie.deltastreamer.keygen.class=org.apache.hudi.keygen.SimpleKeyGenerator

# Storage location
hoodie.base.path=s3a://my-lake/hudi_orders
```

---

#### Step 2: Run DeltaStreamer in Continuous Mode

Use the following command to launch DeltaStreamer in **continuous ingestion** mode:

```bash
spark-submit \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
--master yarn \
--deploy-mode cluster \
/path/to/hudi-utilities-bundle.jar \
--props kafka-source.properties \
--target-base-path s3a://my-lake/hudi_orders \
--table-type COPY_ON_WRITE \
--continuous
```

This runs indefinitely and continuously consumes from Kafka, writing every few seconds based on commit interval.

---

#### Step 3: Tune for Low Latency

- **Reduce commit delay**:
  ```properties
  hoodie.deltastreamer.write.batch.size=5000
  hoodie.write.commit.trigger.strategy=num_commits
  ```

- **Enable async compaction (for MOR)**:
  ```properties
  hoodie.compact.inline=false
  hoodie.compact.async.enabled=true
  ```

- **Enable checkpointing** to ensure resume from last read offset:
  DeltaStreamer uses a `.checkpoint` file stored in the base path automatically.

- **Use Avro with Schema Registry** for schema evolution:
  ```properties
  hoodie.deltastreamer.source.kafka.value.deserializer=io.confluent.kafka.serializers.KafkaAvroDeserializer
  ```

---

#### Step 4: Query the Data

If Hive sync is enabled, the table becomes instantly queryable:

```sql
SELECT * FROM default.hudi_orders
WHERE region = 'us-east' AND event_ts > '2024-11-16T00:00:00';
```

You can use:
- **Presto / Trino**
- **Athena**
- **Hive**
- **Spark SQL**

---

#### Monitoring and Observability

- Use Spark UI to track job progress and memory usage
- Enable Hudi metrics:
  ```
  hoodie.metrics.on=true
  hoodie.metrics.reporter.type=GRAPHITE
  ```
- Inspect commit timeline using `hudi-cli`:
  ```
  hudi-cli > connect --path s3a://my-lake/hudi_orders  
  hudi-cli > show commits
  ```

---

#### Best Practices

- Use **COPY_ON_WRITE** for fast queries and **MERGE_ON_READ** for faster ingestion
- Use **partitioning** (e.g., by region, date) to boost query speed
- Tune Spark executors and write parallelism:
  ```
  --conf spark.executor.memory=4G  
  --conf spark.sql.shuffle.partitions=100
  ```
- Use **Kafka topic compaction** and proper retention settings

---

#### Conclusion

Using **Hudi DeltaStreamer with Kafka** provides a robust, production-grade framework for building **low-latency streaming pipelines** into your data lake. It simplifies ingestion, supports updates and deletes, and enables fast downstream analytics.

With the right configuration and tuning, this architecture supports real-time use cases such as **CDC ingestion**, **event stream processing**, and **operational analytics** on top of cloud-native lakehouse storage.
