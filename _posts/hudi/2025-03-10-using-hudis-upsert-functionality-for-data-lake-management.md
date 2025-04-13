---
layout: post
title: Using Hudi Upsert Functionality for Data Lake Management
subtitle: Manage mutable data efficiently in your data lake with Apache Hudi's upsert capabilities
categories: Hudi
tags: [Hudi, Apache Hudi, Upsert, Data Lake, Big Data, Lakehouse, Data Engineering]
excerpt: Discover how Apache Hudi's upsert functionality enables real-time, ACID-compliant data lake management. Learn how to configure and use upserts to handle mutable data efficiently at scale.
---
Traditional data lakes are designed for **append-only data**, making it difficult to handle **updates and deletes** — a critical need in modern applications like customer tracking, inventory systems, and change data capture (CDC) pipelines.

**Apache Hudi** solves this by enabling **upserts**: the ability to update existing records or insert new ones based on a unique key. This allows data engineers to **build efficient, mutable, and consistent data lakes** on top of cloud-native storage systems like Amazon S3, HDFS, or Azure ADLS.

In this blog, we’ll explore how to use **Hudi’s upsert functionality** for managing mutable data in data lakes, including write operations, configurations, and best practices for scale.

---

#### What is an Upsert?

**Upsert** = **Update** + **Insert**  
An upsert checks if a record already exists:
- If it does, update it.
- If it doesn’t, insert it.

Hudi performs upserts efficiently by maintaining a **record key** and a **precombine field** to determine which version of the record should be retained.

---

#### Why Use Upserts in Data Lakes?

Use cases include:
- **Change Data Capture (CDC)** from databases
- **User profile tracking** (e.g., last login, preferences)
- **Order management systems** (status updates, returns)
- **Inventory updates** (real-time stock adjustments)

Benefits of Hudi upserts:
- **ACID guarantees** even on object storage
- **Time-travel queries** with versioned data
- **Efficient record-level mutation**
- Integration with **Hive**, **Spark**, **Presto**, and **Athena**

---

#### Configuring Hudi for Upserts

Hudi supports two storage types:
- **Copy-on-Write (COW)**: Updates create new versions of files
- **Merge-on-Read (MOR)**: Updates written as delta logs, compacted later

Sample configuration for an upsert job in PySpark:

```python
df.write.format("hudi") \
.option("hoodie.table.name", "customers") \
.option("hoodie.datasource.write.recordkey.field", "customer_id") \
.option("hoodie.datasource.write.precombine.field", "updated_at") \
.option("hoodie.datasource.write.operation", "upsert") \
.option("hoodie.datasource.write.table.name", "customers") \
.option("hoodie.datasource.write.hive_style_partitioning", "true") \
.option("hoodie.datasource.hive_sync.enable", "true") \
.option("hoodie.datasource.hive_sync.mode", "hms") \
.option("hoodie.datasource.hive_sync.database", "default") \
.option("hoodie.datasource.hive_sync.table", "customers") \
.mode("append") \
.save("s3://my-data-lake/hudi/customers")
```

---

#### How Upserts Work in Hudi

1. Each incoming record is matched by its **record key**.
2. If there’s an older version in the dataset:
  - The **precombine field** (e.g., `updated_at`) determines which version to keep.
3. The upserted data is written either:
  - As a new **base file** (COW)
  - As a **log file** (MOR) to be compacted later.

This approach avoids costly full rewrites and supports **incremental data ingestion**.

---

#### Reading Data after Upserts

Use standard read:

```python
spark.read.format("hudi").load("s3://my-data-lake/hudi/customers").show()
```

Or filter with Hive/Presto:

```sql
SELECT * FROM customers WHERE updated_at >= '2024-01-01'
```

Hudi supports **incremental reads**, useful for streaming or micro-batch processing:

```python
spark.read.format("hudi") \
.option("hoodie.datasource.query.type", "incremental") \
.option("hoodie.datasource.read.begin.instanttime", "20240101000000") \
.load("s3://my-data-lake/hudi/customers")
```

---

#### Best Practices for Upserts

- Always define a **unique record key** (e.g., `order_id`, `customer_id`)
- Use a **precombine field** with monotonically increasing timestamps
- Choose **COW** for read-heavy workloads, **MOR** for write-heavy workloads
- Monitor **compaction frequency** in MOR to avoid read performance issues
- Use **Hive sync** to expose tables to Presto, Athena, or HiveServer2
- Schedule **cleaner jobs** to remove old file versions and save storage

---

#### Common Pitfalls to Avoid

- Missing or duplicate record keys can cause silent data inconsistencies
- Improper precombine field selection may keep stale data
- Over-partitioning can lead to small file problems on S3
- Ignoring compaction for MOR tables can slow down reads drastically

---

#### Conclusion

Apache Hudi’s upsert functionality provides a powerful solution for **managing mutable, real-time datasets** in data lakes. With support for **record-level mutation**, **incremental ingestion**, and **Hive integration**, Hudi transforms your data lake into a reliable, queryable, and transaction-safe lakehouse.

By following the configuration strategies and best practices outlined here, you can build scalable and consistent pipelines that support modern analytical and operational workloads.
