---
layout: post
title: Using Hudi with DeltaStreamer for Simplified Data Ingestion
subtitle: Streamline big data ingestion using Apache Hudi’s DeltaStreamer for scalable, incremental pipelines
categories: Hudi
tags: [Hudi, DeltaStreamer, Data Ingestion, Apache Hudi, Big Data, Streaming ETL, Lakehouse]
excerpt: Discover how to use Apache Hudi’s DeltaStreamer for real-time and batch data ingestion. Learn setup, configuration, and best practices for building robust and incremental pipelines.
---
Apache Hudi is a powerful data lakehouse framework that enables upserts, time-travel, and incremental querying on distributed storage systems like HDFS or S3. To simplify the process of ingesting data into Hudi tables, Hudi provides a built-in tool called **DeltaStreamer**.

**Hudi DeltaStreamer** offers a low-code, scalable solution for ingesting **batch and streaming data** from sources such as Kafka, HDFS, and DFS-compatible systems. It automates schema inference, compaction, and syncing with Hive Metastore or AWS Glue Catalog.

In this guide, we’ll explore how to use **DeltaStreamer** to build production-ready ingestion pipelines with minimal code and maximum flexibility.

---

#### What is Hudi DeltaStreamer?

DeltaStreamer is a **command-line utility** bundled with Hudi that allows users to:

- Ingest data from **DFS sources** or **Kafka**
- Perform **upserts, inserts**, and **bulk inserts**
- Support **Copy-on-Write (COW)** and **Merge-on-Read (MOR)** tables
- Sync metadata with Hive or AWS Glue
- Apply basic transformations using SQL or custom hooks

It is ideal for users who don’t want to write Spark jobs manually.

---

#### Supported Source Types

DeltaStreamer supports multiple source types via its **Source classes**:

| Source Type | Description                    |
|-------------|--------------------------------|
| DFS         | Reads from CSV, JSON, Parquet  |
| Kafka       | Supports Avro/JSON messages    |
| Hive        | For SQL-based ingestion        |
| JDBC        | For relational database sources|

For streaming ingestion, **Kafka** is the most common source. For batch, use **DFS**.

---

#### Setting Up Hudi DeltaStreamer

**Prerequisites:**
- Apache Hudi installed (Hudi bundle JAR or tarball)
- Spark 3.x compatible environment
- Source data in S3, HDFS, or Kafka
- Hive Metastore or AWS Glue configured (optional)

**Basic directory layout:**
```
/hudi
├── deltastreamer-config/
├── input-data/
└── scripts/
```

---

#### Example: Batch Ingestion from HDFS (DFS Source)

```bash
spark-submit \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
--master yarn \
/path/to/hudi-utilities-bundle.jar \
--table-type COPY_ON_WRITE \
--source-class org.apache.hudi.utilities.sources.ParquetDFSSource \
--target-base-path hdfs:///user/hudi/orders_cow \
--target-table orders_cow \
--props /hudi/deltastreamer-config/orders.properties \
--schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
--source-ordering-field ts \
--hoodie-conf hoodie.datasource.write.recordkey.field=order_id \
--hoodie-conf hoodie.datasource.write.partitionpath.field=order_date \
--run-bootstrap
```

---

#### Sample Properties File (`orders.properties`)

```properties
hoodie.deltastreamer.source.dfs.root=hdfs:///user/data/orders_raw
hoodie.datasource.write.table.name=orders_cow
hoodie.datasource.hive_sync.enable=true
hoodie.datasource.hive_sync.database=default
hoodie.datasource.hive_sync.table=orders_cow
hoodie.datasource.hive_sync.partition_fields=order_date
hoodie.datasource.hive_sync.use_jdbc=false
hoodie.datasource.hive_sync.mode=hms
```

---

#### Example: Streaming Ingestion from Kafka

```bash
spark-submit \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
--master yarn \
/path/to/hudi-utilities-bundle.jar \
--table-type MERGE_ON_READ \
--source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
--target-base-path s3a://datalake/transactions_mor \
--target-table transactions_mor \
--props /hudi/deltastreamer-config/transactions.properties \
--schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
--continuous
```

This runs a **long-running streaming job** that continuously ingests and merges updates from Kafka.

---

#### Schema Provider Options

- **FileBasedSchemaProvider**: Reads Avro schema from disk
- **ClasspathBasedSchemaProvider**: Reads schema from classpath JAR
- **HiveSchemaProvider**: Pulls schema from Hive table

Use the schema provider that best fits your environment and governance model.

---

#### Running Compaction for MOR Tables

For MOR tables, compaction is needed to merge logs:

```bash
spark-submit \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
--master yarn \
/path/to/hudi-utilities-bundle.jar \
--compact \
--target-base-path s3a://datalake/transactions_mor \
--target-table transactions_mor \
--props /hudi/deltastreamer-config/transactions.properties
```

Alternatively, use **inline compaction** with:

```properties
hoodie.compact.inline=true
hoodie.compact.inline.max.delta.commits=5
```

---

#### Best Practices

- Use **dedicated schema providers** for governance and stability
- Partition on **low-cardinality fields** (e.g., date, region)
- Use **inline compaction** for low-latency use cases
- Enable **Hive sync** for downstream querying (Athena, Hive, Presto)
- Monitor ingestion jobs via **logs and checkpoints**
- For production, integrate with **Airflow**, **Dagster**, or **Glue Workflows**

---

#### Conclusion

**DeltaStreamer** simplifies building streaming and batch ingestion pipelines into Hudi without writing Spark code. Whether you're ingesting from Kafka, S3, or HDFS, DeltaStreamer provides a unified and configurable interface to build **incremental**, **versioned**, and **queryable** data lakes with minimal setup.

By adopting Hudi and DeltaStreamer, you can accelerate the path to a **real-time, scalable, and cloud-native lakehouse** architecture.
