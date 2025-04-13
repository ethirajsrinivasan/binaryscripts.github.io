---
layout: post
title: Building Time Travel Queries with Hudi in Apache Spark
subtitle: Leverage Apache Hudi’s time travel capabilities to query historical data states in Spark
categories: Hudi
tags: [Hudi, Apache Spark, Time Travel, Big Data, Lakehouse, Data Versioning, Data Engineering]
excerpt: Discover how to build time travel queries using Apache Hudi and Spark. Learn how to query data as of a specific point in time or commit version, and apply time travel for auditing, rollback, and historical analytics.
---
Modern data platforms require more than just storage and processing — they demand **version control, auditability, and historical insights**. Apache Hudi addresses these needs with **time travel queries**, allowing users to access data **as it existed at any point in the past**.

With Apache Spark, you can harness Hudi’s time travel features to build powerful data pipelines that support **data auditing**, **rollback**, and **temporal analytics** — all within your existing lakehouse architecture.

This blog explores how to build **time travel queries with Hudi in Apache Spark**, including query examples, configuration tips, and real-world use cases.

---

#### What is Time Travel in Hudi?

Time travel in Hudi enables querying **historical versions of a dataset** based on:

- **Commit timestamp** (e.g., data as of "2024-11-01 10:00:00")
- **Commit ID** (e.g., a specific Hudi write operation)

This is useful for:
- Debugging and auditing
- Comparing dataset versions
- Restoring prior data states
- Recomputing downstream tables

Hudi supports time travel on both **Copy-on-Write (COW)** and **Merge-on-Read (MOR)** tables.

---

#### Enabling Time Travel in Spark with Hudi

Hudi’s time travel feature is available via **Spark SQL**, **DataFrame API**, or the **Hudi CLI**.

To enable time travel, make sure you're using:

- Apache Spark 3.x+
- Apache Hudi 0.10.0+
- Hudi tables stored in S3, HDFS, or any Hadoop-compatible file system

---

#### Querying with Commit Timestamp

You can query data **as of a specific time** using the `as.of.instant` option.

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
.appName("Hudi Time Travel Query") \
.config("spark.serializer", "org.apache.spark.serializer.KryoSerializer") \
.getOrCreate()

# Replace with the actual timestamp or commit time in UTC
timestamp = "2024-11-01T10:00:00.000Z"

df = spark.read.format("hudi") \
.option("as.of.instant", timestamp) \
.load("s3://your-bucket/hudi/orders")

df.show()
```

This returns the snapshot of the Hudi table **as it existed at that timestamp**.

---

#### Querying with Commit ID (Instant Time)

You can also use a specific **commit ID**, which is available via the Hudi timeline.

To get commit history:

```bash
hdfs dfs -ls s3://your-bucket/hudi/orders/.hoodie
# or use Hudi CLI: show commits
```

Query using the commit ID:

```python
df = spark.read.format("hudi") \
.option("as.of.instant", "20241101100000") \
.load("s3://your-bucket/hudi/orders")
```

---

#### Listing All Commits with Spark

You can use Hudi’s timeline API to inspect all historical commits:

```python
from org.apache.hudi import HoodieCLIUtils

meta_client = HoodieCLIUtils.createMetaClient(spark.sparkContext._jsc.hadoopConfiguration(),
"s3://your-bucket/hudi/orders")

instants = meta_client.getCommitsTimeline().getInstants().toArray()
for instant in instants:
print(instant.getTimestamp())
```

This helps in building custom UIs, lineage tools, or rollback pipelines.

---

#### Use Case: Auditing Historical Orders

Imagine needing to audit what orders looked like **before a bug fix** was deployed:

```python
# Query pre-bug-fix data
df_before_fix = spark.read.format("hudi") \
.option("as.of.instant", "20241031110000") \
.load("s3://your-bucket/hudi/orders")

df_before_fix.createOrReplaceTempView("orders_before")

spark.sql("SELECT customer_id, total FROM orders_before WHERE order_status = 'FAILED'").show()
```

---

#### Limitations and Tips

- Time travel **does not work** if commits have been archived
  - Adjust archiving settings:
    ```
    hoodie.cleaner.commits.retained=30
    hoodie.archival.min.commits=20
    hoodie.archival.max.commits=40
    ```
- For MOR tables, ensure **compaction does not remove older versions**
- Time travel is **read-only** — you cannot write to a past snapshot
- Works best with **Hive-style partitioning** for efficient filtering

---

#### Best Practices

- Use **consistent commit times** in UTC format
- Archive commits only after validation or downstream sync
- Use **external metadata tables** (e.g., AWS Glue) to track commit lineage
- Store commit metadata in dashboards or audit logs
- Test time travel queries with Athena or Presto if integrated

---

#### Conclusion

Apache Hudi’s time travel capability is a game-changer for **data governance, debugging, and reproducibility**. By integrating this feature with Apache Spark, teams can confidently audit historical data, troubleshoot pipelines, and build **version-aware** analytics with ease.

Implementing time travel in your data lake not only improves transparency, but also brings your architecture one step closer to a **production-ready lakehouse**.
