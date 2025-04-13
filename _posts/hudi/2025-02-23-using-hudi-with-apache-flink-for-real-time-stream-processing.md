---
layout: post
title: Using Hudi with Apache Flink for Real Time Stream Processing
subtitle: Build real-time data lakes using Apache Hudi with Apache Flink for continuous, low-latency ingestion
categories: Hudi
tags: [Hudi, Apache Flink, Stream Processing, Real-Time, Lakehouse, S3, Big Data]
excerpt: Learn how to integrate Apache Hudi with Apache Flink to build real-time, incremental data lakes. Understand stream write architecture, Flink sink configuration, and best practices for low-latency processing.
---
Modern data platforms increasingly require **real-time ingestion and updates** to support analytics, fraud detection, user personalization, and more. While Apache Hudi provides **incremental data lake capabilities**, pairing it with a stream processing engine like **Apache Flink** unlocks **true real-time streaming ingestion** with guaranteed upserts and efficient writes to data lakes.

In this blog, we'll explore how to use **Apache Hudi with Apache Flink**, covering stream write architecture, Flink sink configuration, and real-world best practices for **building a low-latency lakehouse architecture**.

---

#### Why Flink + Hudi?

Apache Flink is a **high-throughput, low-latency** stream processing framework with support for event-time semantics, windowing, and exactly-once guarantees. Integrating it with Hudi allows you to:

- Ingest **change data capture (CDC)** streams from Kafka or Debezium
- Perform **real-time upserts/deletes** to S3 or HDFS
- Maintain **transactional, queryable tables** for downstream consumption (e.g., Athena, Hive, Presto)
- Enable **time-travel queries** and **incremental processing**

---

#### Architecture Overview

```
Kafka / CDC / Logs  
↓  
[Flink Source]  
↓  
[Flink → Hudi Sink]  
↓  
S3 / HDFS  
↓  
Query via Presto / Athena / Hive / Flink SQL  
```

Flink reads real-time events, applies processing logic, and writes the output directly to Hudi in **streaming write mode**.

---

#### Supported Table Types and Modes

Flink supports writing to both **Copy-on-Write (COW)** and **Merge-on-Read (MOR)** tables.

- Use **COW** for fast read/query performance
- Use **MOR** for low-latency writes and compaction optimization

Table types are specified in the job properties and determine how the data is physically stored and queried.

---

#### Setting Up the Flink Hudi Sink

**Prerequisites:**
- Apache Flink 1.14 or later
- Apache Hudi 0.10.0+ (with flink-bundle JAR)
- Access to a storage layer (e.g., S3, HDFS)

**Flink SQL DDL Example:**

```sql
CREATE TABLE hudi_orders (
order_id STRING PRIMARY KEY NOT ENFORCED,
customer_id STRING,
amount DOUBLE,
order_ts TIMESTAMP(3),
partition_date STRING
)
PARTITIONED BY (partition_date)
WITH (
'connector' = 'hudi',
'path' = 's3a://my-lake/orders',
'table.type' = 'MERGE_ON_READ',
'hoodie.datasource.write.recordkey.field' = 'order_id',
'write.precombine.field' = 'order_ts',
'write.tasks' = '4',
'compaction.async.enabled' = 'true',
'compaction.delta_commits' = '5',
'hive_sync.enable' = 'true',
'hive_sync.mode' = 'hms',
'hive_sync.database' = 'default',
'hive_sync.table' = 'hudi_orders',
'hive_sync.metastore.uris' = 'thrift://hive-metastore:9083'
);
```

You can then stream into this table with:

```sql
INSERT INTO hudi_orders
SELECT
order_id, customer_id, amount, order_ts,
DATE_FORMAT(order_ts, 'yyyy-MM-dd') AS partition_date
FROM kafka_orders_stream;
```

---

#### Flink Job Configuration (Java/PyFlink)

If you're using the **DataStream API**, use the `FlinkStreamer` utility from the Hudi distribution or implement a custom job using:

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

DataStream<RowData> sourceStream = ...

Configuration conf = new Configuration();
conf.setString(FlinkOptions.PATH, "s3a://my-lake/orders");
conf.setString(FlinkOptions.TABLE_TYPE, TableType.MERGE_ON_READ.name());
conf.setString(FlinkOptions.RECORD_KEY_FIELD, "order_id");
conf.setString(FlinkOptions.PRECOMBINE_FIELD, "order_ts");

HoodieTableSink sink = new HoodieTableSink(conf);
sourceStream.sinkTo(sink);
```

---

#### Performance and Tuning Tips

- **Use async compaction** to prevent blocking streaming jobs
  ```
  'compaction.async.enabled' = 'true'
  ```

- **Adjust parallelism** based on input rate:
  ```
  'write.tasks' = '4'
  ```

- **Avoid small files** with proper sizing configs:
  ```
  'hoodie.parquet.small.file.limit' = '104857600'
  ```

- **Enable checkpointing** in Flink for fault tolerance:
  ```java
  env.enableCheckpointing(60000); // every 60 seconds
  ```

- Use **Hive Catalog sync** for schema management across tools

---

#### Monitoring and Validation

- Monitor metrics via Flink’s Web UI or Prometheus
- Use `hudi-cli` or `HoodieTimeline` API to inspect commit timelines
- Query data with Athena or Presto to validate freshness and correctness

```sql
SELECT * FROM hudi_orders WHERE partition_date = '2024-11-16';
```

---

#### Use Cases in Production

- **Change Data Capture (CDC)** from transactional databases using Debezium → Flink → Hudi
- **Clickstream ingestion** for near real-time dashboards
- **IoT event tracking** with high-frequency device streams
- **Fraud detection pipelines** that require millisecond-latency inserts and lookups

---

#### Conclusion

Integrating **Apache Hudi with Apache Flink** bridges the gap between **streaming ingestion** and **lakehouse storage**. It enables real-time updates to S3/HDFS-backed data lakes with support for upserts, schema evolution, and queryable data — all while maintaining transactional integrity.

If you're building **real-time data lakes** or modernizing legacy ETL pipelines, this integration offers a powerful and production-ready foundation.
