---
layout: post
title: Integrating Hudi with Elasticsearch for Data Searchability
subtitle: Enable fast, flexible search on your Hudi lakehouse data by syncing it to Elasticsearch
categories: Hudi
tags: [Hudi, Elasticsearch, Data Search, Big Data, Lakehouse, Apache Hudi, Indexing, Real-time Analytics]
excerpt: Learn how to integrate Apache Hudi with Elasticsearch to power full-text search, real-time analytics, and metadata indexing for your data lakehouse. Includes setup, sync strategies, and best practices.
---
While **Apache Hudi** is designed for scalable data ingestion, updates, and time-travel in data lakes, it’s not built for **search-first use cases** like full-text lookup or metadata indexing. That’s where **Elasticsearch** comes in.

By integrating Hudi with Elasticsearch, you get the best of both worlds:
- Hudi for **transactional data lake storage**
- Elasticsearch for **searchable, real-time indexing**

In this guide, you'll learn how to sync Hudi data to Elasticsearch, enabling fast and flexible search on your lakehouse data for downstream applications and dashboards.

---

#### Why Integrate Hudi with Elasticsearch?

| Feature             | Hudi                            | Elasticsearch                    |
|---------------------|----------------------------------|----------------------------------|
| Storage Layer       | Transactional data lake (S3, HDFS) | Distributed search index         |
| Query Use Case      | SQL analytics, batch, time-travel | Full-text search, filter, real-time |
| Consistency Model   | ACID (Copy-on-Write or MOR)       | Eventually consistent             |
| Integration Value   | Durable ingestion and versioning | Fast retrieval and filtering      |

Use cases:
- **Searchable logs or events** from data lake
- **Metadata search** on schema catalogs
- **Real-time indexing** of transactional updates
- **Analytics dashboards** powered by Kibana

---

#### Architecture Overview

The integration architecture looks like this:

```
[Kafka/CDC/Streaming] → [Hudi Table] → [Hudi to ES Sync Job] → [Elasticsearch Index] → [Search/UI]
```

The core component is the **Hudi to Elasticsearch sync job**, which extracts updated records and pushes them into ES indices.

---

#### Sync Methods

There are several ways to sync data from Hudi to Elasticsearch:

1. **DeltaStreamer with Elasticsearch Sink**
2. **Custom Spark job using Hudi Incremental Read + ES output**
3. **Logstash or Fluentd pipeline for post-ingestion sync**

---

#### Method 1: Using Hudi DeltaStreamer with Elasticsearch Sink

Hudi includes a built-in **Elasticsearch sink** for `DeltaStreamer`.

**Step 1: Set up Hudi DeltaStreamer**

```bash
hudi-utilities-bundle.jar \
--target-base-path s3://my-lake/hudi-logs \
--target-table hudi_log_data \
--table-type COPY_ON_WRITE \
--source-class org.apache.hudi.utilities.sources.JsonDFSSource \
--source-ordering-field log_ts \
--hoodie-conf hoodie.datasource.write.recordkey.field=log_id \
--hoodie-conf hoodie.deltastreamer.sink.class=org.apache.hudi.elasticsearch.ElasticsearchSink \
--hoodie-conf hoodie.embed.timeline.server=true \
--hoodie-conf hoodie.elasticsearch.url=http://localhost:9200 \
--hoodie-conf hoodie.elasticsearch.index=hudi_logs
```

**Step 2: Verify in Elasticsearch**

Use Kibana or:

```bash
curl http://localhost:9200/hudi_logs/_search?q=level:ERROR
```

---

#### Method 2: Custom Spark Job for Incremental Sync

Use Spark to read only the new Hudi records and push to Elasticsearch.

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
.appName("hudi-es-sync") \
.config("spark.serializer", "org.apache.spark.serializer.KryoSerializer") \
.getOrCreate()

# Define Hudi read options
hudi_options = {
'hoodie.datasource.query.type': 'incremental',
'hoodie.datasource.read.begin.instanttime': '20240415010101',
'hoodie.datasource.read.end.instanttime': '20240416010101'
}

df = spark.read.format("hudi") \
.options(**hudi_options) \
.load("s3://my-lake/hudi/logs")

# Write to Elasticsearch
df.write \
.format("org.elasticsearch.spark.sql") \
.option("es.nodes", "localhost") \
.option("es.port", "9200") \
.option("es.resource", "hudi_logs/_doc") \
.mode("append") \
.save()
```

Make sure to update `begin.instanttime` from Hudi commit timeline regularly.

---

#### Schema Mapping Tips

- Flatten nested JSON in Spark before writing to ES
- Ensure `recordkey` is mapped to Elasticsearch `_id` for upserts
- Avoid highly dynamic fields — define an index template for structure

---

#### Indexing Strategy

- Use **date-based indices** for time-series logs (e.g., `hudi_logs_2024_04_16`)
- Apply **retention policies** in Elasticsearch to delete old data
- Enable **refresh_interval=30s** to balance index freshness vs performance
- Set **number_of_shards** based on query volume and index size

---

#### Monitoring and Troubleshooting

Monitor:

- Hudi write and commit timelines
- Elasticsearch index growth and health (`_cat/indices`)
- Sync job latency and failures

Tools:
- **Kibana dashboards**
- **Elastic Stack alerting**
- **Spark logs + audit**

---

#### Best Practices

- Use **Incremental Queries** from Hudi for efficiency
- Enable **async compaction** in Hudi to avoid delays in data availability
- Compress data using **Snappy + Parquet** in Hudi
- Batch writes to Elasticsearch for performance (`bulk.enabled=true`)
- Monitor **_id collisions** in Elasticsearch due to poor record key choice

---

#### Conclusion

Integrating **Hudi with Elasticsearch** bridges the gap between **transactional lakehouse storage** and **search-optimized indexing**. Whether you're building a searchable log store, real-time dashboard backend, or metadata discovery platform, this architecture brings high-performance and flexibility to your data lake ecosystem.

By choosing the right sync strategy and following best practices, you can deliver **scalable, queryable, and near-real-time access** to Hudi-managed data through Elasticsearch.
