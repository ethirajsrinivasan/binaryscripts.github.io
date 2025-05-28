---
layout: post
title: Advanced Monitoring of Pulsar Clusters with Prometheus and Grafana
subtitle: Gain deep operational visibility into Apache Pulsar clusters using Prometheus and Grafana
categories: Pulsar
tags: [Pulsar, Monitoring, Prometheus, Grafana, Observability, Metrics, Alerting, Apache Pulsar]
excerpt: Learn how to monitor Apache Pulsar clusters with Prometheus and Grafana. Track broker, topic, and subscription metrics to ensure performance, detect bottlenecks, and build alert-driven observability systems.
---
Monitoring is essential for managing the health, performance, and scalability of production-grade Apache Pulsar clusters. As workloads grow and systems become more distributed, the need for **real-time observability** becomes paramount.

In this post, we’ll explore how to set up **advanced monitoring of Apache Pulsar using Prometheus and Grafana**, providing insights into **broker health**, **topic throughput**, **latency**, **consumer lag**, and more.

---

#### Why Use Prometheus and Grafana?

**Prometheus** is an open-source systems monitoring and alerting toolkit that excels at scraping metrics exposed via HTTP.

**Grafana** is a powerful visualization platform that integrates with Prometheus to create dashboards, alerts, and reports.

Together, they offer:

- Real-time metrics collection
- Customizable visual dashboards
- Alerting on threshold breaches
- Historical trend analysis

---

#### Exposing Pulsar Metrics to Prometheus

Pulsar exposes **JMX metrics** and optionally **Prometheus-format metrics** via HTTP.

To enable Prometheus exporter:

##### 1. Start Pulsar with Prometheus enabled

Edit `broker.conf` and `bookkeeper.conf`:

```
metricsProviderClassName=org.apache.pulsar.broker.stats.prometheus.PrometheusMetricsServlet
```

Pulsar brokers expose metrics at:

```
http://<broker-host>:8080/metrics
```

Bookies expose metrics at:

```
http://<bookie-host>:8000/metrics
```

---

#### Setting Up Prometheus

##### 1. Install Prometheus (via binary or Docker)

##### 2. Configure `prometheus.yml`:

```yaml
scrape_configs:
- job_name: 'pulsar-brokers'
  static_configs:
  - targets: ['broker1:8080', 'broker2:8080']

- job_name: 'pulsar-bookies'
  static_configs:
  - targets: ['bookie1:8000', 'bookie2:8000']
    ```

Start Prometheus:

```bash
./prometheus --config.file=prometheus.yml
```

---

#### Setting Up Grafana

1. Install Grafana and launch the web UI (default port: `3000`)

2. Add **Prometheus** as a data source:
  - URL: `http://localhost:9090`
  - Access: Server

3. Import Pulsar dashboards:
  - Use [Grafana dashboards for Pulsar](https://grafana.com/grafana/dashboards)
  - Or create custom panels with Prometheus queries

---

#### Key Pulsar Metrics to Monitor

| Category      | Metric Name                                | Description                                      |
|---------------|---------------------------------------------|--------------------------------------------------|
| Broker Load   | `pulsar_broker_cpu_usage`                  | CPU usage of broker                              |
|               | `pulsar_broker_active_connections`         | Number of active client connections              |
| Topic         | `pulsar_topic_msg_rate_in`                 | Message ingestion rate                           |
|               | `pulsar_topic_msg_rate_out`                | Message delivery rate                            |
|               | `pulsar_topic_backlog`                     | Messages not yet consumed                        |
| Subscription  | `pulsar_subscription_msg_backlog`          | Number of pending messages for a subscription    |
|               | `pulsar_subscription_msg_ack_rate`         | Rate at which messages are acknowledged          |
| BookKeeper    | `bookkeeper_journal_add_entry_latency`     | Latency of write operations to journal           |
|               | `bookkeeper_ledger_write_latency`          | Ledger persistence latency                       |

Use these metrics to:
- Identify bottlenecks
- Track message lag and consumption patterns
- Alert on broker unavailability or overload
- Monitor end-to-end message latency

---

#### Sample Prometheus Queries

- **Total message rate in:**

```promql
sum(pulsar_topic_msg_rate_in)
```

- **Consumer lag by subscription:**

```promql
pulsar_subscription_msg_backlog{subscription="my-sub"}
```

- **Broker CPU usage:**

```promql
pulsar_broker_cpu_usage
```

- **Messages per topic:**

```promql
topk(10, pulsar_topic_msg_rate_in)
```

---

#### Alerts and Notifications

Define Prometheus alert rules in `alert.rules.yml`:

```yaml
groups:
- name: pulsar-alerts
  rules:
  - alert: HighBrokerCPU
    expr: pulsar_broker_cpu_usage > 0.85
    for: 2m
    labels:
    severity: warning
    annotations:
    summary: "High CPU usage on Pulsar broker"
    description: "CPU usage is above 85% for more than 2 minutes."
    ```

Connect Grafana to **email**, **Slack**, or **PagerDuty** to send real-time alerts.

---

#### Best Practices

- Scrape metrics every 10–15 seconds for near-real-time visibility
- Label metrics by **cluster, tenant, namespace** for filtering
- Use **recording rules** for expensive queries
- Group dashboards by role: Broker Ops, Tenant Usage, Topic Health
- Regularly audit unused topics with zero message flow
- Combine with **OpenTelemetry** for trace-level observability

---

#### Conclusion

Apache Pulsar offers robust monitoring hooks that, when combined with **Prometheus and Grafana**, provide full-stack visibility into your messaging infrastructure. From broker health to topic throughput and consumer lag, these tools help you build **scalable**, **observable**, and **resilient** Pulsar deployments.

By adopting these practices, you ensure that your Pulsar platform remains **performant, well-managed, and alert-ready** — whether running in the cloud, on-prem, or across multiple regions.
