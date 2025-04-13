---
layout: post
title: Leveraging Hudi with Presto for Interactive Queries
subtitle: Enable fast, interactive analytics on Apache Hudi tables using Presto or Trino
categories: Hudi
tags: [Hudi, Presto, Trino, Big Data, Interactive Queries, Lakehouse, Analytics]
excerpt: Learn how to query Apache Hudi tables with Presto for low-latency, interactive analytics. Explore setup, table formats, query modes, and performance tuning for Hudi-Presto integration.
---
Apache Hudi brings ACID transactions and incremental data capabilities to data lakes, while **Presto (or Trino)** provides a distributed SQL engine for fast, interactive analytics. Together, they form a powerful stack for building **real-time, queryable lakehouses**.

In this post, we’ll explore how to **leverage Hudi with Presto** for running interactive queries on large datasets. You’ll learn about supported table formats, read modes, Hive Metastore integration, and key performance optimizations for low-latency analytics.

---

#### Why Use Presto with Hudi?

Combining Presto and Hudi offers several advantages:

- **Fast query execution** using MPP (Massively Parallel Processing)
- **ACID-compliant datasets** managed by Hudi
- **Schema evolution and upserts** with Hudi, seamlessly queried via Presto
- **Federated access** to Hive, Kafka, S3, and more

Presto supports reading both **Copy-on-Write (COW)** and **Merge-on-Read (MOR)** Hudi tables via the Hive connector.

---

#### Table Formats in Hudi

Hudi supports two main table types:

| Type           | Storage Format | Query Speed | Write Speed | Use Case                         |
|----------------|----------------|-------------|-------------|----------------------------------|
| Copy-on-Write  | Parquet        | Fast        | Slower      | Read-heavy, batch analytics      |
| Merge-on-Read  | Parquet + Avro | Medium      | Faster      | Write-heavy, streaming ingestion |

Presto can query both, but **COW** tables deliver **better performance** for ad-hoc BI queries.

---

#### Prerequisites

Before querying Hudi tables with Presto:

1. **Hudi tables must be Hive-synced**
2. **Hive Metastore must be accessible to Presto**
3. **Presto Hive connector** must be configured correctly

Update `catalog/hive.properties` in Presto:

```properties
connector.name=hive-hadoop2
hive.metastore.uri=thrift://<hive-metastore-host>:9083
hive.allow-drop-table=true
hive.non-managed-table-writes-enabled=true
hive.parquet.use-column-names=true
```

Ensure Presto has permissions to access the Hudi table storage location (e.g., HDFS, S3).

---

#### Creating and Syncing Hudi Tables

Example: Creating a Hudi COW table and syncing it with Hive.

```scala
df.write
.format("hudi")
.option("hoodie.datasource.write.recordkey.field", "user_id")
.option("hoodie.datasource.write.table.name", "user_events")
.option("hoodie.datasource.write.operation", "upsert")
.option("hoodie.datasource.hive_sync.enable", "true")
.option("hoodie.datasource.hive_sync.database", "analytics")
.option("hoodie.datasource.hive_sync.table", "user_events")
.option("hoodie.datasource.hive_sync.mode", "hms")
.mode("append")
.save("/warehouse/analytics/user_events")
```

Once synced, Presto can access the table via:

```sql
SELECT * FROM hive.analytics.user_events WHERE event_type = 'login' LIMIT 50;
```

---

#### Querying Merge-on-Read Tables

By default, Presto reads the **latest snapshot** of MOR tables.

To query the **base files only** (like a COW table), configure read mode in your query engine or catalog properties.

Caveats when using MOR tables:
- Slower performance for deep compaction or large delta logs
- Queries may not reflect latest writes unless **compaction** is triggered

For best performance:
- Compact the table regularly using Hudi CLI or Spark job
- Query COW tables for dashboards and ad-hoc analytics

---

#### Performance Optimization Tips

1. **Use Partition Pruning**  
   Structure tables with partition fields (e.g., `event_date`) and filter accordingly:

   ```sql
   SELECT * FROM user_events WHERE event_date = '2024-11-15';
   ```

2. **Avoid SELECT * on wide tables**  
   Project only the necessary columns.

3. **Enable vectorized reads** in Presto's Hive connector config:

   ```properties
   hive.parquet.reader.enabled=true
   hive.parquet.use-column-names=true
   ```

4. **Compact MOR tables periodically**  
   Helps reduce log file reads and improves latency.

   ```bash
   hudi-cli
   connect --path /warehouse/analytics/user_events
   compaction schedule
   compaction run
   ```

5. **Use Presto Caching** (if supported in your distribution)

---

#### Use Cases for Hudi + Presto

- **Real-time dashboards**: Combine Hudi's streaming ingest with Presto's fast queries
- **Data lake exploration**: Query historical + incremental data without ETL
- **Data quality audits**: Run anomaly detection on recent ingested records
- **Federated analytics**: Join Hudi data with external RDBMS, Kafka, or JSON logs

---

#### Conclusion

Combining **Apache Hudi and Presto** offers a flexible, high-performance solution for modern data lakes. Hudi ensures scalable, ACID-compliant data ingestion, while Presto enables **low-latency querying** for analytics, dashboards, and ad-hoc exploration.

By understanding Hudi’s storage models, syncing tables with Hive, and tuning your Presto setup, you can build a powerful, interactive analytics stack that bridges the gap between **real-time data ingestion** and **enterprise-scale querying**.
