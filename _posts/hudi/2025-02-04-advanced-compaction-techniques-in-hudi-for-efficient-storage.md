---
layout: post
title: Advanced Compaction Techniques in Hudi for Efficient Storage
subtitle: Master Hudi compaction strategies to optimize storage, reduce latency, and maintain data lake performance
categories: Hudi
tags: [Apache Hudi, Compaction, Big Data, Lakehouse, Storage Optimization, Incremental Processing]
excerpt: Discover advanced compaction techniques in Apache Hudi to manage storage efficiently and speed up queries. Learn about inline, asynchronous, and clustering-based compaction strategies for modern data lakes.
---
**Apache Hudi (Hadoop Upserts Deletes and Incrementals)** is a powerful lakehouse storage engine that enables real-time ingestion, updates, deletes, and incremental queries on data lakes. As data evolves rapidly, especially with frequent inserts and updates, Hudi uses **compaction** to merge smaller delta files into optimized base files.

Efficient compaction is crucial for maintaining **query performance**, **reducing small files**, and **lowering storage overhead**. In this blog, we explore **advanced compaction techniques in Hudi**, including inline compaction, asynchronous compaction, and clustering for efficient storage management.

---

#### Why Is Compaction Important in Hudi?

In **Copy-on-Write (CoW)** tables, data is written as new base files. In **Merge-on-Read (MoR)** tables, updates are written as delta logs and periodically compacted into base files. Without compaction:

- MoR queries degrade over time (many logs to merge)
- Small file problems increase NameNode pressure
- Query latency increases due to excessive merging at read time

Efficient compaction ensures:
- **Faster read performance**
- **Reduced file count**
- **Efficient use of HDFS or cloud storage**

---

#### Hudi Table Types Recap

1. **Copy-on-Write (CoW)**:
  - Updates rewrite entire files
  - No compaction needed
  - Better for read-heavy use cases

2. **Merge-on-Read (MoR)**:
  - Writes deltas as log files
  - Requires periodic compaction
  - Ideal for write-heavy workloads with streaming updates

---

#### Inline Compaction

**Inline compaction** performs compaction immediately after writes as part of the ingestion job.

**Enable inline compaction:**

```json
hoodie.compact.inline=true
hoodie.compact.inline.max.delta.commits=3
```

Pros:
- Automatic and convenient
- Reduces read amplification quickly

Cons:
- Slower ingestion due to added compaction step
- May impact latency for real-time pipelines

Use inline compaction when:
- Data volumes are manageable
- You prefer simplicity over fine-tuned scheduling

---

#### Asynchronous (Scheduled) Compaction

In **asynchronous compaction**, ingestion is separated from compaction using background jobs.

**Steps:**
1. Ingest data normally
2. Run compaction periodically as a separate job

```bash
spark-submit \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
... \
--run-bootstrap --op compact
```

Or using CLI:

```bash
hudi-cli
connect --path s3://datalake/hudi/orders
compactions schedule
compactions run
```

Pros:
- No impact on write latency
- Full control over compaction timing

Cons:
- More operational overhead
- Requires job orchestration (Airflow, Oozie, etc.)

Use this for large-scale streaming ingestion and when **latency is critical**.

---

#### Compaction Trigger Strategies

Tune compaction frequency using:

```json
hoodie.compact.inline.max.delta.commits=5
hoodie.compaction.small.file.size=104857600  # 100 MB
hoodie.io.compaction.strategy.class=org.apache.hudi.io.compact.strategy.UnboundedIOCompactionStrategy
```

Other strategies include:
- **BoundedIOCompactionStrategy**: Limits file size per compaction batch
- **DayBasedCompactionStrategy**: Compacts based on partition time range
- **LogFileSizeBasedStrategy**: Triggers based on accumulated log file size

Choose a strategy based on ingestion rate, partitioning scheme, and query latency SLAs.

---

#### Compaction with Clustering (Hudi 0.7+)

**Clustering** is a layout optimization that reorganizes data files based on sorting and sizing.

Use clustering to:
- Sort data by frequently queried columns
- Coalesce small files across partitions

Enable clustering:

```json
hoodie.clustering.inline=true
hoodie.clustering.inline.max.commits=4
hoodie.clustering.strategy.class=org.apache.hudi.client.clustering.plan.strategy.SparkSortAndSizeExecutionStrategy
```

Schedule and run clustering separately:

```bash
clustering schedule
clustering run
```

Use clustering when:
- Query performance is more critical than ingest speed
- You experience skewed or fragmented data

---

#### Best Practices for Efficient Compaction

- Use **asynchronous compaction** for high-throughput streaming ingestion
- Limit file size to reduce I/O overhead (`hoodie.parquet.max.file.size`)
- Set **log file size thresholds** to trigger compaction timely
- Combine **clustering with compaction** to improve read locality
- Monitor compaction lag via Hudi metadata and dashboards
- Schedule compaction during **low-traffic hours**

---

#### Monitoring and Metrics

Track compaction metrics using:
- Hudi CLI (`show compaction` commands)
- Spark job logs
- Hudi metadata tables (`.hoodie/` directory)
- Integrations with Prometheus, Grafana, or Hadoop UIs

Focus on:
- Number of pending compactions
- Average compaction duration
- File sizes before and after compaction

---

#### Conclusion

Apache Hudi’s compaction framework is essential for maintaining **efficient storage**, **low-latency queries**, and **manageable file systems** in modern data lakes. By using the right combination of **inline**, **asynchronous**, and **clustering-based compaction techniques**, you can balance write throughput and read performance effectively.

Master these advanced compaction techniques to ensure your Hudi-powered lakehouse remains **fast, scalable, and cost-efficient** as your data grows.
