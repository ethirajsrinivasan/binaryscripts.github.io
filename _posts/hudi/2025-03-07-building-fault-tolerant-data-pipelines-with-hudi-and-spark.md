---
layout: post
title: Building Fault Tolerant Data Pipelines with Hudi and Spark
subtitle: Design resilient and scalable data pipelines using Apache Hudi and Apache Spark
categories: Hudi
tags: [Hudi, Apache Spark, Data Pipelines, Fault Tolerance, Lakehouse, Big Data, Streaming]
excerpt: Learn how to build fault-tolerant and scalable data pipelines using Apache Hudi and Spark. Explore checkpointing, schema handling, transactional writes, and recovery strategies for robust big data workflows.
---
In today’s data-driven world, building **fault-tolerant data pipelines** is essential for ensuring **data reliability**, **accuracy**, and **availability** in production environments. Apache Hudi, combined with Apache Spark, offers a powerful framework to build resilient pipelines with **ACID guarantees**, **incremental processing**, and **support for recovery from failures**.

In this blog post, we'll explore how to design and implement **fault-tolerant data pipelines using Hudi and Spark**, and highlight key features, configurations, and recovery strategies to build enterprise-grade lakehouse ingestion systems.

---

#### Why Hudi and Spark for Data Pipelines?

Apache Hudi is designed for **streaming ingestion and transactional storage** on data lakes like S3 and HDFS, while Apache Spark provides distributed compute capabilities.

Together, they offer:

- **Atomic writes** to distributed storage
- Support for **upserts, deletes, and incremental loads**
- **Write recovery** using commit timeline
- **Scalable ETL** using batch or streaming Spark jobs

This makes them ideal for handling **real-time CDC**, **IoT feeds**, **log ingestion**, and **data warehouse offloads**.

---

#### Pipeline Architecture Overview

```
[Kafka / Kinesis / CDC Logs]  
↓  
[Spark Structured Streaming]  
↓  
[Apache Hudi Table (COW/MOR)] on [S3 / HDFS]  
↓  
[Glue Catalog / Hive Metastore]  
↓  
[Athena / Trino / Presto / Spark SQL]  
```

Each layer is responsible for ingestion, transformation, transactional persistence, and queryability — with **fault tolerance built-in at every step**.

---

#### Key Features for Fault Tolerance

1. **Hudi Commit Timeline**
  - Every write to a Hudi table is recorded in a timeline (e.g., `instant time`)
  - Enables **rollback**, **compaction tracking**, and **time travel**

2. **Transactional Writes**
  - Ensures **atomicity**, so partial writes don’t corrupt downstream reads
  - Supports **idempotent writes** with precombine keys and deduplication

3. **Checkpoints in Spark Streaming**
  - Spark can recover from crashes using checkpoints and offsets

```python
query = df.writeStream \
.format("hudi") \
.option("checkpointLocation", "s3://checkpoints/hudi/stream1/") \
.outputMode("append") \
.start("s3://datalake/hudi/orders/")
```

---

#### Configuring Hudi for Reliable Writes

Example Spark write config for a fault-tolerant ingestion:

```python
hudi_options = {
'hoodie.table.name': 'orders_hudi',
'hoodie.datasource.write.recordkey.field': 'order_id',
'hoodie.datasource.write.precombine.field': 'ts',
'hoodie.datasource.write.partitionpath.field': 'order_date',
'hoodie.datasource.write.operation': 'upsert',
'hoodie.datasource.write.table.type': 'MERGE_ON_READ',
'hoodie.cleaner.policy': 'KEEP_LATEST_COMMITS',
'hoodie.keep.max.commits': 20,
'hoodie.keep.min.commits': 10,
'hoodie.datasource.write.keygenerator.class': 'org.apache.hudi.keygen.ComplexKeyGenerator'
}
```

Key points:
- **MERGE_ON_READ** enables high-throughput writes with lazy compaction
- Cleaner policies ensure metadata doesn’t grow unbounded
- Precombine field ensures latest record is retained in case of duplicates

---

#### Handling Schema Evolution

Apache Hudi supports **automatic schema evolution** across versions:

- Schema is tracked in Avro format via the table metadata
- Enable schema validation:

```
hoodie.avro.schema.validate=true
hoodie.datasource.write.schema.evolution.enable=true
```

Best practice:
- Always define a **compatible schema** with nullable fields for future additions

---

#### Compaction and Clustering Resilience

For **Merge-on-Read** tables, compaction is essential for performance:

- Use **inline compaction** for small-scale jobs:

```
hoodie.compact.inline=true  
hoodie.compact.inline.max.delta.commits=5  
```

- Or use **asynchronous compaction** in production:

```
hoodie.compact.async=true  
```

Both modes are **fault-tolerant** and recoverable through timeline tracking.

---

#### Recovery from Write Failures

If a Spark job fails mid-write:

1. The **commit file is not created** (Hudi uses "marker" files)
2. The **next job rolls back** incomplete commits
3. Use the CLI to list and roll back manually if needed:

```bash
hudi-cli
> connect --path s3://my-hudi-table
> show commmits
> rollback <instant_time>
```

---

#### Monitoring and Observability

Use the following to monitor pipeline health:

- **Hudi Timeline Server UI**
- **Apache Spark UI** for job and task performance
- **CloudWatch / Prometheus** for S3 or job metrics
- Audit commit metadata with:

```bash
hdfs dfs -cat s3://table/.hoodie/ .commit files
```

---

#### Best Practices

- Enable **checkpointing** and use **idempotent writes**
- Tune **compaction** and **cleaner policies** for long-term stability
- Use **precombine keys** to avoid stale records
- Validate schema compatibility before writing
- Monitor **commit timeline** regularly for orphaned or inflight commits

---

#### Conclusion

Apache Hudi and Spark together provide a powerful foundation for building **fault-tolerant, real-time data pipelines**. With support for incremental writes, transactional guarantees, and recovery mechanisms, Hudi helps ensure that your big data platform is not only scalable — but also **resilient and production-grade**.

By applying the right configurations and monitoring strategies, you can build **durable, efficient, and low-latency data workflows** that support a wide range of business-critical applications.
