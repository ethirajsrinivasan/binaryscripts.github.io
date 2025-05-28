---
layout: post
title: Monitoring Kafka Clusters with Prometheus and Grafana
subtitle: Set up end-to-end observability for Apache Kafka using Prometheus and Grafana
categories: Kafka
tags: [Kafka, Prometheus, Grafana, Monitoring, Observability, DevOps, Metrics, Big Data]
excerpt: Learn how to monitor Apache Kafka clusters using Prometheus and Grafana. This guide covers JMX Exporter setup, essential metrics, Grafana dashboard configuration, and best practices for real-time observability.
---
Apache Kafka powers mission-critical streaming platforms, which makes **monitoring and observability essential** to maintain uptime, performance, and reliability. Without proper monitoring, diagnosing issues like **lag**, **under-replicated partitions**, or **broker failures** becomes extremely difficult.

This blog will guide you through setting up **Prometheus and Grafana** to monitor Kafka clusters effectively. You’ll learn how to export Kafka metrics using JMX Exporter, configure Prometheus scraping, visualize key indicators in Grafana, and apply alerting best practices.

---

#### Architecture Overview

```
[Kafka Brokers] → [JMX Exporter] → [Prometheus] → [Grafana Dashboards]
↓
[AlertManager]
```

Kafka exposes metrics via **Java Management Extensions (JMX)**. These metrics are scraped by **Prometheus** using the **JMX Exporter agent**, stored as time-series data, and visualized using **Grafana**.

---

#### Step 1: Set Up JMX Exporter for Kafka

Download the JMX Exporter agent JAR:

```bash
wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.18.0/jmx_prometheus_javaagent-0.18.0.jar
```

Create a Kafka metrics config file (e.g., `kafka-jmx-config.yaml`):

```yaml
startDelaySeconds: 0
lowercaseOutputName: true
rules:
- pattern: 'kafka.server<type=(.+), name=(.+)><>Value'
  name: kafka_$1_$2
  type: GAUGE
  help: "Kafka metric $1 $2"
  ```

Update Kafka startup script (`KAFKA_HEAP_OPTS`) to include:

```
-javaagent:/opt/jmx_exporter/jmx_prometheus_javaagent-0.18.0.jar=7071:/opt/jmx_exporter/kafka-jmx-config.yaml
```

This exposes metrics on `http://<broker-ip>:7071/metrics`.

---

#### Step 2: Configure Prometheus to Scrape Kafka Metrics

Edit `prometheus.yml` to add a Kafka scrape job:

```yaml
scrape_configs:
- job_name: 'kafka-brokers'
  static_configs:
  - targets: ['kafka-broker1:7071', 'kafka-broker2:7071']
    ```

Start Prometheus:

```bash
./prometheus --config.file=prometheus.yml
```

---

#### Step 3: Import Kafka Dashboard into Grafana

1. Start Grafana and connect it to Prometheus as a data source.
2. Import an existing Kafka dashboard from [Grafana's dashboard hub](https://grafana.com/grafana/dashboards/721) or create your own.
3. Key visualizations to include:
  - Broker and Topic throughput (messages in/out per second)
  - Partition count per topic
  - Under-replicated partitions
  - Consumer group lag
  - Disk usage and JVM heap stats

---

#### Essential Kafka Metrics to Monitor

| Category              | Metric                                 | Description                          |
|-----------------------|-----------------------------------------|--------------------------------------|
| **Broker Health**     | kafka_server_broker_topic_metrics      | Messages in/out                      |
| **Replication**       | kafka_server_replicamanager_underreplicatedpartitions | Shows unreplicated partitions        |
| **Consumer Lag**      | kafka_consumergroup_lag                | Measures lag per group/topic/partition |
| **Requests**          | kafka_network_requestmetrics_totaltimems | Tracks latency of produce/fetch APIs |
| **Heap Memory**       | jvm_memory_bytes_used                  | Monitors memory usage                |

---

#### Setting Up Alerting with Prometheus Alertmanager

Define alerting rules in `prometheus.yml`:

```yaml
groups:
- name: kafka_alerts
  rules:
  - alert: KafkaUnderReplicatedPartitions
    expr: kafka_server_replicamanager_underreplicatedpartitions > 0
    for: 5m
    labels:
    severity: warning
    annotations:
    summary: "Under-replicated partitions detected"
    ```

Send alerts via:
- Email
- Slack
- Webhooks
- PagerDuty

---

#### Best Practices for Kafka Monitoring

- Monitor **all brokers** individually and collectively
- Track **consumer lag** per group, topic, and partition
- Watch for **increased GC activity or heap pressure**
- Use **Grafana annotations** to mark deploys or outages
- Set up alerts for:
  - Broker downtime
  - Message rate drops
  - Replication issues
  - Disk threshold violations

---

#### Conclusion

Monitoring Kafka with **Prometheus and Grafana** gives you deep visibility into your cluster’s health and performance. From broker throughput to consumer lag, this observability stack ensures your event streaming platform remains **resilient, fast, and production-ready**.

Implement these practices to catch issues early, scale with confidence, and deliver real-time data services with peace of mind.
