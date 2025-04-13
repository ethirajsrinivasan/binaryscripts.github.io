---
layout: post
title: Debugging Hudi Compactions Tips and Tricks
subtitle: Master the art of troubleshooting compaction issues in Apache Hudi Merge-on-Read tables
categories: Hudi
tags: [Hudi, Compaction, Merge-on-Read, Debugging, Big Data, Performance, Lakehouse]
excerpt: Learn how to debug and troubleshoot Apache Hudi compaction issues. Understand compaction architecture, view commit timelines, handle failures, and optimize compaction for better performance and consistency.
---
Apache Hudi’s **Merge-on-Read (MOR)** tables offer powerful real-time ingestion and upsert capabilities. However, these benefits come with the additional complexity of managing **compaction** — the process of merging delta logs into base files for optimized reads.

If you're working with large-scale streaming data or building near real-time lakehouse architectures, it's essential to know how to **debug compaction issues**, recover from failures, and tune for performance.

This post shares **tips and tricks for debugging Hudi compactions**, including tooling, timeline inspection, Spark configurations, and recovery strategies.

---

#### What Is Compaction in Hudi?

In **MOR tables**, incoming data is written as **delta log files** (e.g., `.log`) for quick ingestion. Periodically, these logs are merged with base files (`.parquet`) through **compaction**.

Benefits of compaction:
- Reduces query latency
- Improves predicate pushdown and scan efficiency
- Consolidates small files

Compaction can be **inline** (during writes) or **asynchronous** (background job).

---

#### Detecting Compaction Issues

Common symptoms of compaction problems:
- Slow queries over MOR tables
- Large number of delta log files
- Failed or stuck compaction operations
- Incomplete data showing in readers (e.g., Hive, Presto)

Use the **Hudi Timeline** to diagnose:

```bash
hdfs dfs -ls s3://my-lake/.hoodie/
```

Look for `.inflight`, `.requested`, `.compaction` files.

---

#### Tip 1: View Compaction Status with Hudi CLI

The **Hudi CLI** is your best friend for investigating compaction timelines.

```bash
hudi-cli
> connect --path s3://my-lake/orders
> compactions show all
> compactions show pending
> compactions show inflight
```

Check specific compaction details:

```
> compaction show files --compactionInstant 20241116084523
```

This shows which file groups and partitions are affected.

---

#### Tip 2: Run Manual or Async Compaction

To manually compact MOR tables:

```bash
spark-submit \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
--master yarn \
--deploy-mode cluster \
--conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer' \
--conf 'spark.sql.shuffle.partitions=100' \
--conf 'hoodie.compact.inline=false' \
--conf 'hoodie.compact.schedule.inline=false' \
--conf 'hoodie.compact.inline.max.delta.commits=5' \
--conf 'hoodie.compaction.async.enable=true' \
$HUDI_JAR \
--run-compaction --base-path s3://my-lake/orders --table-type MERGE_ON_READ
```

Set the right value for `hoodie.compact.inline.max.delta.commits` to control how frequently compaction is triggered.

---

#### Tip 3: Inspect Commit Timeline

Use this to identify if compactions are stuck or skipped:

```bash
> commits show all
> commits show archived
> show fsview all
```

Look for:
- Missing `.compaction` entries
- `.inflight` compaction that never completes
- Long gaps between delta commits and compaction commits

---

#### Tip 4: Clean Up Incomplete Compactions

If compaction is stuck in `.inflight` or `.requested` state, remove it manually **only if it's safe**:

```bash
hdfs dfs -rm s3://my-lake/.hoodie/20241116084523.compaction.inflight
```

Then re-run compaction. Make sure the table is **not being written to concurrently** during this cleanup.

Alternatively, use:

```
> compaction unschedule --compactionInstant 20241116084523
```

---

#### Tip 5: Tune Compaction Performance

Use these configs to speed up compactions:

```properties
hoodie.compaction.async.enable=true
hoodie.compact.inline=false
hoodie.compact.inline.max.delta.commits=10
hoodie.compaction.payload.class=org.apache.hudi.common.model.OverwriteWithLatestAvroPayload
hoodie.parquet.max.file.size=134217728
hoodie.compaction.small.file.limit=104857600
hoodie.compaction.strategy.class=org.apache.hudi.table.action.compact.strategy.UnBoundedCompactionStrategy
```

Increase `spark.executor.memory` and `spark.executor.cores` to allocate more resources to compaction.

---

#### Tip 6: Monitor with Metrics and Logs

Enable Hudi metrics for compaction via:

```properties
hoodie.metrics.on=true
hoodie.metrics.reporter.type=CONSOLE
hoodie.write.status.storage.level=DISK
```

You can also integrate with Prometheus/Grafana by exporting metrics from the Spark executor or Hudi client.

Review Spark UI for long stages or failed tasks.

---

#### Tip 7: Compact by Partition (Advanced)

To compact specific partitions only:

```bash
> compactions schedule --partitions "2024/11/15,2024/11/16"
> compactions run --parallelism 8
```

Useful for targeted fixes where specific partitions have excessive log files.

---

#### Conclusion

**Hudi compaction is critical** for maintaining read performance and query efficiency in Merge-on-Read tables. Whether you’re troubleshooting inflight commits, optimizing compaction intervals, or tuning Spark configurations, these techniques will help you build a robust and performant data lakehouse with Apache Hudi.

Use the **Hudi CLI**, monitor timelines, and automate compaction intelligently to stay ahead of growing data volumes.
