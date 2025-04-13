---
layout: post
title: Optimizing Hudi Queries with Spark SQL
subtitle: Speed up analytics on Hudi datasets using Spark SQL with partition pruning, predicate pushdown, and index tuning
categories: Hudi
tags: [Hudi, Spark SQL, Performance, Query Optimization, Lakehouse, Big Data, Incremental Processing]
excerpt: Learn best practices for optimizing Apache Hudi queries using Spark SQL. Improve performance with partition pruning, metadata tuning, column stats, and incremental read strategies.
---
Apache Hudi is a powerful lakehouse platform that supports efficient upserts, time-travel, and incremental processing. But to truly unlock the potential of Hudi for **fast analytics**, you need to understand how to optimize queries — especially when using **Spark SQL**.

In this guide, we’ll explore strategies to **optimize Hudi queries using Spark SQL**, including tips for partition pruning, file indexing, metadata use, and incremental reads.

---

#### 1. Enable Hudi-Specific Optimizations

Before running any queries, make sure Spark is configured to work well with Hudi:

```scala
spark.conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
spark.conf.set("spark.sql.hive.convertMetastoreParquet", "false")
spark.conf.set("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.hudi.catalog.HoodieCatalog")
```

These settings ensure that Spark interacts properly with Hudi tables and doesn't treat them like regular Parquet files.

---

#### 2. Leverage Partition Pruning

Hudi supports Hive-style partitioning, which Spark SQL can utilize to **skip unnecessary file scans**.

**Example partitioned table:**

```sql
CREATE TABLE hudi_sales (
id STRING,
product STRING,
amount DOUBLE,
sale_date STRING
)
USING hudi
PARTITIONED BY (sale_date)
OPTIONS (
type = 'cow',
primaryKey = 'id',
preCombineField = 'sale_ts'
);
```

**Optimized query:**

```sql
SELECT * FROM hudi_sales WHERE sale_date = '2024-11-16';
```

Avoid using functions on partition columns (e.g., `DATE(sale_date)`) in WHERE clauses, as this disables pruning.

---

#### 3. Enable Predicate Pushdown and Column Pruning

Use **Parquet** and **ORC** file formats with **Snappy** compression for optimal query performance. These formats support:

- Predicate pushdown: filters rows early during scan
- Column pruning: reads only required columns

```sql
SELECT amount FROM hudi_sales WHERE product = 'Laptop';
```

This query only scans the `product` and `amount` columns — reducing I/O.

---

#### 4. Use Incremental Queries

Instead of scanning the entire dataset, Spark can read only **new or updated records** since the last commit.

```scala
val commits = spark.sql("SELECT DISTINCT(_hoodie_commit_time) FROM hudi_sales ORDER BY _hoodie_commit_time DESC LIMIT 2")
val beginTime = commits.collect().last.getString(0)

val df = spark.read.format("hudi")
.option("hoodie.datasource.query.type", "incremental")
.option("hoodie.datasource.read.begin.instanttime", beginTime)
.load("s3://my-hudi/sales")

df.show()
```

This reduces the cost of repeat analytics jobs and supports CDC pipelines.

---

#### 5. Use Hudi Metadata Table

Enable the metadata table for faster file listing:

```properties
hoodie.metadata.enable=true
hoodie.metadata.index.column.stats.enable=true
hoodie.metadata.partition.stats.enable=true
```

Metadata table eliminates the need for expensive file listing on large datasets — a key boost in cloud environments like S3 or GCS.

---

#### 6. Optimize with Column Statistics and Bloom Index

Enable **column stats index** to accelerate filter queries:

```properties
hoodie.metadata.index.column.stats.enable=true
```

Enable **bloom index** to reduce file lookup time for point queries:

```properties
hoodie.index.type=BLOOM
hoodie.bloom.index.filter.type=MERGE
hoodie.bloom.index.update.partition.path=true
```

This is especially useful for workloads with selective filters on primary keys.

---

#### 7. Compact and Clean Regularly

For **Merge-on-Read (MOR)** tables, compaction improves query performance:

```bash
spark-submit \
--class org.apache.hudi.utilities.HoodieCompactor \
--master yarn \
$HUDI_UTILITIES_JAR \
--base-path s3://my-hudi/sales \
--table-name hudi_sales
```

Also, use **clean** operations to remove obsolete files and reduce storage bloat.

---

#### 8. Tune Spark and Hudi Parameters

For large-scale reads:

```scala
spark.conf.set("spark.sql.shuffle.partitions", "200")
spark.conf.set("spark.sql.files.maxPartitionBytes", "134217728")  // 128MB
spark.conf.set("spark.sql.broadcastTimeout", "600")
```

Set Hudi-specific read options:

```scala
.option("hoodie.datasource.read.as.of.instant", "20241116153000")  // time-travel
.option("hoodie.datasource.read.streaming.skip_compaction", "true")
```

---

#### 9. Query via Spark Catalog or Hive

If you registered your Hudi table in Hive Metastore or Glue Catalog:

```sql
SELECT * FROM spark_catalog.default.hudi_sales WHERE sale_date = '2024-11-16';
```

Ensure Hudi tables are synced using HiveSync:

```properties
hoodie.datasource.hive_sync.enable=true
hoodie.datasource.hive_sync.mode=hms
hoodie.datasource.hive_sync.database=default
hoodie.datasource.hive_sync.table=hudi_sales
```

---

#### Conclusion

Optimizing Apache Hudi queries using Spark SQL is crucial for maintaining performance in large-scale data lakes. By leveraging **partition pruning**, **predicate pushdown**, **metadata acceleration**, and **incremental reads**, you can significantly reduce query latency and processing time.

These strategies help build **scalable, cost-efficient** lakehouse pipelines ready for modern analytics and machine learning workloads.
