---
layout: post
title: Using Spring Boot with Apache Hudi for Big Data Applications
subtitle: Build scalable, incremental data pipelines with Spring Boot and Apache Hudi
categories: SpringBoot
tags: [Spring Boot, Apache Hudi, Big Data, Hadoop, Data Lake, Java, Incremental Processing]
excerpt: Learn how to integrate Spring Boot with Apache Hudi to build real-time big data applications. This guide explores Hudi architecture, ingestion patterns, and how to manage large-scale datasets efficiently.
---
In the world of **big data**, the ability to process and update large volumes of data in near real time is critical. Traditional batch pipelines fall short when it comes to **incremental ingestion**, **upserts**, and **low-latency analytics**.

Enter **Apache Hudi** — a data lake storage layer built for **streaming and batch workloads**. It provides support for ACID transactions, record-level updates, and integration with Spark, Hive, and Presto.

This post explores how to use **Spring Boot** in conjunction with Apache Hudi to create big data pipelines that are scalable, efficient, and cloud-ready.

---

#### What is Apache Hudi?

**Apache Hudi (Hadoop Upserts Deletes and Incrementals)** is an open-source framework that sits on top of HDFS-compatible storage (HDFS, S3, GCS). It enables:
- Insert, update, and delete support for data lakes
- Efficient data compaction and clustering
- Time travel queries using snapshots
- Integration with Spark, Hive, and Presto/Trino

Hudi supports two storage types:
- **Copy on Write (CoW)**: Safe and optimized for read-heavy workloads
- **Merge on Read (MoR)**: Efficient for frequent writes and updates

---

#### Why Combine Spring Boot and Apache Hudi?

Spring Boot can be used as:
- A **data ingestion microservice** into Hudi
- A **controller layer** that triggers Spark jobs
- A REST API to expose Hudi-managed datasets to downstream systems
- A metadata service layer managing ingestion pipelines

Use cases include:
- CDC ingestion from RDBMS into Hudi
- Real-time data APIs backed by a Hudi data lake
- Orchestration layer for batch compaction and clustering

---

#### Setting Up Your Spring Boot Project

Add necessary dependencies in `pom.xml`:

```xml
<dependency>
<groupId>org.apache.hudi</groupId>
<artifactId>hudi-spark3.3-bundle_2.12</artifactId>
<version>0.14.0</version>
</dependency>
<dependency>
<groupId>org.apache.spark</groupId>
<artifactId>spark-core_2.12</artifactId>
<version>3.3.2</version>
</dependency>
```

Use the **Hudi Spark bundle** to programmatically write data from your Spring service via a Spark driver.

---

#### Writing to Apache Hudi with SparkSession

Inside your Spring Boot service, use Spark to write Parquet files to Hudi:

```java
SparkConf conf = new SparkConf()
.setAppName("spring-hudi-writer")
.setMaster("local[*]");

SparkSession spark = SparkSession.builder()
.config(conf)
.getOrCreate();

Dataset<Row> inputData = spark.read().json("data/input.json");

inputData.write()
.format("hudi")
.options(Map.of(
"hoodie.table.name", "hudi_users",
"hoodie.datasource.write.recordkey.field", "id",
"hoodie.datasource.write.partitionpath.field", "country",
"hoodie.datasource.write.precombine.field", "timestamp",
"hoodie.datasource.write.operation", "upsert",
"hoodie.datasource.write.table.type", "COPY_ON_WRITE",
"hoodie.datasource.write.hive_style_partitioning", "true"
))
.mode(SaveMode.Append)
.save("hdfs://namenode:9000/user/hudi/hudi_users");
```

This performs **upserts** into a Hudi table, partitioned by `country`.

---

#### Triggering Hudi Jobs from REST Endpoints

You can expose REST endpoints in Spring Boot that trigger ingestion jobs:

```java
@RestController
@RequestMapping("/ingest")
public class IngestionController {

    @Autowired
    private SparkJobService sparkJobService;

    @PostMapping
    public ResponseEntity<?> ingestData() {
        sparkJobService.runHudiJob();
        return ResponseEntity.ok("Ingestion started");
    }
}
```

Use `ProcessBuilder` to launch Spark jobs, or embed Spark if running locally for POCs.

---

#### Managing Compaction and Clustering

Over time, Hudi tables accumulate file versions. Use **compaction** to merge delta logs for MoR tables, and **clustering** to optimize file sizes.

Schedule them using Spring Scheduler or Apache Airflow:

```bash
spark-submit \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
--conf ... \
deltastreamer.jar \
--run-compaction
```

Use REST APIs to trigger compaction based on system load or time intervals.

---

#### Querying Hudi Tables

Hudi tables are queryable via:
- **Apache Hive** (sync supported out-of-the-box)
- **Presto/Trino**
- **Apache Spark SQL**

Example Spark SQL query:

```sql
SELECT * FROM hudi_users WHERE country = 'US'
```

Hudi supports snapshot queries, read-optimized, and incremental modes.

---

#### Security and Performance Tips

- Use `Merge on Read` for frequent updates, `Copy on Write` for analytics
- Compress data with `ZSTD` for performance
- Secure HDFS/S3 paths using IAM, Kerberos, or Ranger
- Store metadata and timestamps for CDC-friendly ingestion
- Tune parallelism and memory in Spark for large datasets

---

#### Conclusion

Combining **Spring Boot** and **Apache Hudi** opens up new possibilities in building **modern data pipelines**. With Spring handling orchestration and APIs, and Hudi managing efficient, incremental data storage, you get the best of both real-time and batch processing.

Whether you're processing IoT telemetry, customer transactions, or machine logs, this stack enables you to build **scalable, reliable, and cloud-native big data applications**.
