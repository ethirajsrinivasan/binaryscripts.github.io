---
layout: post
title: Best Practices for Scaling Apache Hudi in Production
subtitle: Optimize performance, reliability, and cost when scaling Apache Hudi for large-scale data lakes
categories: Hudi
tags: [Hudi, Apache Hudi, Data Lake, Scaling, Performance, Production, Big Data]
excerpt: Learn the best practices for deploying and scaling Apache Hudi in production environments. Explore performance tuning, compaction strategies, storage formats, metadata optimization, and operational tips.
---
Apache Hudi brings powerful transactional capabilities to modern data lakes, enabling real-time ingestion, updates, and time-travel querying over immutable cloud storage like Amazon S3, HDFS, or Azure ADLS. But successfully scaling Hudi in a **production environment** requires thoughtful architecture, resource management, and configuration tuning.

This guide outlines **best practices for scaling Apache Hudi in production**, covering ingestion performance, metadata management, compaction optimization, and reliability strategies to build robust lakehouse platforms at scale.

---

#### 1. Choose the Right Table Type: COW vs MOR

| Table Type         | When to Use                                    |
|--------------------|------------------------------------------------|
| Copy-on-Write (COW)| Frequent reads, low update rates               |
| Merge-on-Read (MOR)| High ingestion rate, frequent upserts/deletes  |

- Use **COW** for Athena, Presto, and low-latency querying
- Use **MOR** for streaming pipelines, high-write workloads, and large updates

---

#### 2. Optimize Partitioning Strategy

- Use **low-cardinality** columns like `date`, `region`, or `event_type`
- Avoid over-partitioning (e.g., using `user_id` or `UUID`)
- Enable **Hive-style partitioning**:

```
hoodie.datasource.write.hive_style_partitioning=true
```

Partition pruning improves query speed and reduces scan overhead.

---

#### 3. Tune File Sizes to Avoid Small File Problems

- Set minimum file size limits:
  ```
  hoodie.parquet.small.file.limit=134217728  # 128MB
  ```

- Enable file size-based clustering:
  ```
  hoodie.clustering.plan.strategy.class=org.apache.hudi.client.clustering.plan.strategy.SparkSizeBasedClusteringPlanStrategy
  ```

- Cluster periodically or inline:
  ```
  hoodie.clustering.inline=true
  hoodie.clustering.inline.max.commits=4
  ```

---

#### 4. Manage Compaction and Cleaning Efficiently

**For MOR tables**:
- Use **asynchronous compaction** in high-throughput scenarios:
  ```
  hoodie.compact.inline=false
  hoodie.compact.schedule.inline=true
  ```

- Tune delta commits threshold:
  ```
  hoodie.compact.max.delta.commits=5
  ```

**For both COW and MOR**:
- Clean older versions and rollback files:
  ```
  hoodie.cleaner.policy=KEEP_LATEST_COMMITS
  hoodie.cleaner.commits.retained=10
  ```

---

#### 5. Enable and Maintain the Metadata Table

The metadata table speeds up file listing and compaction by avoiding direct storage scans.

Enable metadata:

```
hoodie.metadata.enable=true
hoodie.metadata.compact.inline=true
hoodie.metadata.cleaner.policy=KEEP_LATEST_COMMITS
```

Monitor metadata table health with the Hudi CLI or logs.

---

#### 6. Leverage Incremental Querying

Use **incremental pull mode** to improve ETL efficiency:

```python
spark.read.format("hudi") \
.option("hoodie.datasource.query.type", "incremental") \
.option("hoodie.datasource.read.begin.instanttime", "20240401000000") \
.load("s3://data-lake/hudi/orders")
```

- Track last commit time in metadata store
- Use in streaming or DAG-based workflows (Airflow, Spark, Glue)

---

#### 7. Scale Write Throughput with Parallelism

- Increase Spark parallelism settings:

```
spark.sql.shuffle.partitions=400
hoodie.bulkinsert.shuffle.parallelism=400
hoodie.upsert.shuffle.parallelism=400
```

- Use `bulk_insert` for initial loads or backfills

---

#### 8. Monitor, Audit, and Debug Effectively

- Use **Hudi CLI** for inspecting timelines, commits, compactions, and rollbacks
- Export Hudi metrics via **Prometheus** and visualize in **Grafana**
- Track:
  - `commit.duration`
  - `upsert.records.count`
  - `compaction.duration`
  - `metadata.table.size`

Set up alerts for:
- Failed commits
- Lagging compactions
- Growing number of small files

---

#### 9. Ensure Compatibility with Downstream Query Engines

- Use **Hive sync** to register tables in AWS Glue, HMS, or external catalogs:

```
hoodie.datasource.hive_sync.enable=true
hoodie.datasource.hive_sync.mode=glue
hoodie.datasource.hive_sync.partition_fields=event_date
```

- Use **COW tables** for Athena/Presto; **MOR** requires snapshot readers or compaction

---

#### 10. Secure and Govern Your Data Lake

- Enable **AWS IAM or Ranger**-based access control
- Enable **audit logging** via Spark/Glue logs and SIEM integrations
- Mask or encrypt sensitive columns before writing to Hudi

Use tags or metadata (e.g., JSON schema with classifications) to enforce governance policies.

---

#### Conclusion

Scaling Apache Hudi in production is not just about ingesting data faster—it's about **maintaining consistency**, **ensuring reliability**, and **optimizing performance** across workloads. With the right configurations, monitoring tools, and architecture choices, Hudi enables modern data lakehouses to handle **billions of records**, support **real-time updates**, and serve **critical analytics** with confidence.

Apply these best practices to ensure your Hudi pipelines remain **resilient, efficient, and production-grade** as your data footprint grows.
