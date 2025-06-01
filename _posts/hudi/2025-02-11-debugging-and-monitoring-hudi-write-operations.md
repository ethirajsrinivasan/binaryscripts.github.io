---
layout: post
title: Debugging and Monitoring Hudi Write Operations
subtitle: Learn how to track, debug, and optimize Apache Hudi write operations for reliable data ingestion
categories: Hudi
tags: [Hudi, Apache Hudi, Debugging, Monitoring, Data Ingestion, Big Data, Write Operations, Spark]
excerpt: Explore techniques to debug and monitor write operations in Apache Hudi. Learn how to analyze commit timelines, validate metadata, use metrics and logs, and ensure reliable data ingestion at scale.
---
Apache Hudi enables **streaming data ingestion** and **incremental processing** by supporting upserts, deletes, and merges on large datasets. But with great flexibility comes complexity — and **debugging Hudi write operations** is essential for ensuring data accuracy, consistency, and performance in production.

In this guide, we'll explore how to **debug and monitor Hudi write operations**, covering:
- Timeline and commit inspection
- Metadata validation
- Spark logs and metrics
- Hudi CLI tools
- Integration with monitoring systems

---
Hudi supports several write operations:
- **UPSERT** – Update if key exists, insert if not
- **INSERT** – Append-only
- **BULK_INSERT** – High-throughput initial loads
- **DELETE** – Remove records by key

Each write produces a **commit**, represented in the **.hoodie/ directory** as a timeline event:
- `.commit` – Successful commit
- `.inflight` – Ongoing write
- `.requested` – Scheduled but not yet started

Monitoring these artifacts is the key to understanding Hudi's internal state.

---

#### Step 1: Inspecting the Hudi Timeline

Every Hudi table maintains a **timeline** of operations under the `.hoodie/` folder at the base path.

List the timeline:

```bash
ls /data/hudi/user_events/.hoodie/
20240416091523.commit
20240416093000.inflight
20240416094500.requested
```

Use the **Hudi CLI** to interactively inspect this timeline:

```bash
hudi-cli
connect --path /data/hudi/user_events
show commits
show inflights
show fsview all
```

This helps identify failed writes or long-running inflight commits.

---

#### Step 2: Validating Metadata and Partitions

To ensure that a write completed successfully:
- Verify the presence of the expected **.parquet** or **.log** files
- Check that **record keys** were correctly assigned
- Use the CLI to inspect file views and partitions:

```bash
show files
desc partition <partition_path>
```

Additionally, query the table directly:

```scala
val df = spark.read.format("hudi").load("/data/hudi/user_events")
df.groupBy("partition_path").count().show()
```

---

#### Step 3: Monitoring Write Metrics

Hudi exposes runtime metrics that can be tracked using Spark or external tools like Prometheus:

Enable metrics in the write config:

```scala
.option("hoodie.metrics.on", "true")
.option("hoodie.metrics.reporter.type", "JMX")
.option("hoodie.write.status.storage.level", "WRITE_STATUS")
```

Key metrics:
- Number of inserts/updates
- Write amplification (total bytes written)
- Error rate during write
- Time spent in different phases (indexing, compaction, commit)

Use `jconsole` or Prometheus exporters to visualize trends.

---

#### Step 4: Analyzing Spark Logs and Exceptions

Since Hudi relies heavily on **Spark**, write operations are distributed. Check Spark logs for:

- **Task failures** (due to schema mismatches, null primary keys)
- **HoodieWriteConflictException**
- **OutOfMemory errors**
- **Retries or timeouts** during compaction or index lookup

Set Spark log level to DEBUG:

```bash
--conf "spark.driver.extraJavaOptions=-Dlog4j.configuration=log4j-debug.properties"
--conf "spark.executor.extraJavaOptions=-Dlog4j.configuration=log4j-debug.properties"
```

Enable Hudi-specific logs:

```properties
log4j.logger.org.apache.hudi=DEBUG
log4j.logger.org.apache.hudi.io=DEBUG
```

---

#### Step 5: Validating Write Success

A successful write will:
- Generate a **.commit** file in `.hoodie/`
- Populate data files in the proper partition
- Register metadata updates in Hive (if sync is enabled)
- Include a `_SUCCESS` marker (optional, depending on job type)

You can validate programmatically:

```scala
val commits = spark.read.format("hudi")
.option("hoodie.datasource.query.type", "incremental")
.option("hoodie.datasource.read.begin.instanttime", "000")
.load("/data/hudi/user_events")
.select("commit_time")
.distinct()

commits.show()
```

---

#### Step 6: Using Hudi Metrics Dashboard

Hudi integrates with **Prometheus + Grafana** for centralized monitoring. Metrics include:

- Write throughput (records/sec)
- Compaction latency
- Delta commit frequency
- Failed operations

To enable:

```scala
.option("hoodie.metrics.reporter.type", "PROMETHEUS_PUSHGATEWAY")
.option("hoodie.metrics.prometheus.pushgateway.address", "<host>:9091")
.option("hoodie.metrics.prometheus.pushgateway.jobname", "hudi_write_job")
```

Then create Grafana dashboards based on Prometheus queries.

---

#### Step 7: Debugging Common Write Issues

| Issue                          | Resolution                                                                 |
|--------------------------------|----------------------------------------------------------------------------|
| Missing commit file            | Retry job or clean inflight commits via CLI                                |
| HoodieWriteConflictException   | Enable optimistic concurrency controls and check writer locks              |
| Null record key                | Ensure `hoodie.datasource.write.recordkey.field` is properly defined       |
| Hive sync not reflecting data  | Check `hoodie.sync.*` configs and Metastore connectivity                   |
| Compaction stuck or slow       | Review logs and enable async compaction                                   |

Use `hudi-cli`:

```bash
repair rollback --commit 20240416093000
```

To roll back incomplete or corrupt commits.

---

#### Conclusion

Monitoring and debugging Hudi write operations is crucial for building resilient and high-throughput data pipelines. By leveraging the timeline, inspecting metadata, enabling metrics, and using Hudi's CLI and Spark logs, you gain full visibility into ingestion health and performance.

Whether you're working on streaming ETL or near-real-time analytics, mastering these tools ensures **consistent, observable, and production-ready data lakes with Hudi**.
