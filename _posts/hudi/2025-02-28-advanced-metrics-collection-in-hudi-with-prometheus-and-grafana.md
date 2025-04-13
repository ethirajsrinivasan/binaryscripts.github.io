---
layout: post
title: Advanced Metrics Collection in Hudi with Prometheus and Grafana
subtitle: Monitor and visualize Apache Hudi pipelines using Prometheus and Grafana for better observability
categories: Hudi
tags: [Hudi, Metrics, Monitoring, Prometheus, Grafana, Big Data, Lakehouse, Observability]
excerpt: Learn how to collect, expose, and visualize Apache Hudi metrics using Prometheus and Grafana. Get insights into write performance, compaction health, and query latency across your data lakehouse.
---
As Apache Hudi becomes a cornerstone for data lakes and lakehouse architectures, **observability** is essential for operating it reliably at scale. While Hudi provides logs and command-line tools, deep insights into its **internal performance** — such as write throughput, compaction health, and metadata operations — require a proper metrics system.

In this post, you'll learn how to integrate **Apache Hudi with Prometheus and Grafana** to collect, scrape, and visualize advanced metrics. We'll explore how Hudi exposes JVM and application-level metrics and how to build rich dashboards for monitoring and alerting.

---

#### Why Monitor Hudi?

Key reasons to monitor Hudi:

- Track **write and upsert throughput**
- Analyze **compaction lag and frequency**
- Understand **metadata table behavior**
- Detect **pipeline slowdowns or failures**
- Observe **query performance over time**

Metrics provide real-time observability, proactive alerting, and historical analysis for capacity planning and SLA enforcement.

---

#### Metrics Architecture Overview

```
[Hudi Write Job (Spark)] → [Dropwizard Metrics] → [Prometheus Exporter] → [Prometheus] → [Grafana]
```

Hudi uses **Dropwizard Metrics** under the hood, and can expose them through:
- Console
- File
- JMX
- Prometheus HTTP endpoint (recommended)

---

#### Step 1: Enable Prometheus Metrics in Hudi

You can expose metrics via HTTP using the `PrometheusReporter`.

Set these configs in your Hudi job (e.g., in DeltaStreamer, Spark, or PySpark):

```properties
hoodie.metrics.on=true
hoodie.metrics.reporter.type=Prometheus
hoodie.metrics.prometheus.port=9090
hoodie.metrics.prometheus.prefix=hudi_
hoodie.metrics.prometheus.host=0.0.0.0
```

If you're using Spark:

```bash
--conf hoodie.metrics.on=true \
--conf hoodie.metrics.reporter.type=Prometheus \
--conf hoodie.metrics.prometheus.port=9090
```

This exposes metrics on `http://<executor_host>:9090/metrics`

---

#### Step 2: Set Up Prometheus to Scrape Hudi

In `prometheus.yml`, add a job to scrape the Hudi metrics endpoint:

```yml
scrape_configs:
- job_name: 'hudi'
  static_configs:
  - targets: ['hudi-worker-node:9090']
    ```

Make sure Prometheus can reach all Spark executors/workers that expose metrics.

---

#### Step 3: Visualize Metrics in Grafana

1. Launch Grafana and add Prometheus as a data source
2. Create dashboards using Hudi metrics (prefixed by `hudi_`)
3. Example metrics to track:
  - `hudi_write_commit_duration`
  - `hudi_upsert_count`
  - `hudi_compaction_duration`
  - `hudi_metadata_table_size`
  - `hudi_delta_commits_since_last_compaction`

4. Create alert rules in Grafana to trigger when:
  - Write latency exceeds thresholds
  - Compaction backlog grows
  - Job success rate drops

---

#### Sample Grafana Dashboard Panels

- **Write Throughput**
  - Metric: `rate(hudi_upsert_count[1m])`
  - Unit: records/second

- **Compaction Health**
  - Metric: `hudi_delta_commits_since_last_compaction`
  - Use thresholds to warn if compaction is lagging

- **Commit Latency**
  - Metric: `hudi_write_commit_duration`
  - Visualization: heatmap or time series

- **Metadata Size Trend**
  - Metric: `hudi_metadata_table_size`
  - Correlate with number of partitions/files

---

#### Advanced Use Cases

- Integrate with **Alertmanager** to send alerts to Slack, PagerDuty, or email
- Use **labels** for different Hudi tables or environments (`env=prod`, `table=orders`)
- Store long-term metrics in **Thanos** or **Cortex** for year-over-year performance analysis

---

#### Best Practices

- Always expose metrics from both **driver** and **executor** when possible
- Use **dedicated ports** to avoid conflicts in multi-job clusters
- Avoid exposing metrics over public networks — secure using firewalls or reverse proxies
- Use **templated dashboards** in Grafana to filter by Hudi table or job ID

---

#### Conclusion

Monitoring Hudi with **Prometheus and Grafana** brings transparency and control to your data pipelines. Whether you're debugging slow writes, scaling your compaction strategy, or enforcing SLAs, having rich metrics at your fingertips is key to running a production-grade lakehouse.

By following the setup and best practices outlined here, you can ensure **end-to-end observability** across your Hudi-powered data lake.
