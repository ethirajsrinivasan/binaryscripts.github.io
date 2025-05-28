---
layout: post
title: Building a Scalable Log Aggregation System with Pulsar and Elasticsearch
subtitle: Use Apache Pulsar and Elasticsearch to build a real-time, scalable log aggregation pipeline
categories: Pulsar
tags: [Pulsar, Elasticsearch, Log Aggregation, Observability, Streaming, Big Data, Real-Time Analytics]
excerpt: Learn how to design and deploy a scalable log aggregation pipeline using Apache Pulsar and Elasticsearch. Explore architecture, connector setup, and best practices for real-time log monitoring and analysis.
---
As distributed systems grow in complexity, managing and analyzing logs becomes a crucial part of maintaining **observability**, **performance**, and **security**. Traditional logging approaches often fail to scale with modern cloud-native workloads.

Enter **Apache Pulsar** and **Elasticsearch** — a powerful duo for building real-time log aggregation systems that can handle massive throughput while offering fast search and analytics.

In this blog, we’ll show how to build a **scalable log aggregation pipeline** using Pulsar as the **log ingestion platform** and Elasticsearch as the **search and visualization engine**.

---

#### Why Pulsar and Elasticsearch?

**Apache Pulsar**:
- Pub-sub messaging with high throughput
- Multi-tenancy and geo-replication
- Tiered storage and backpressure handling
- Native support for queue and topic-based ingestion

**Elasticsearch**:
- Distributed search engine
- Real-time log indexing and analytics
- Kibana integration for visualization
- JSON-friendly schema-less design

Together, they provide a pipeline that is:
- Scalable and resilient
- Real-time and searchable
- Cloud-native and extensible

---

#### Architecture Overview

```
[Applications / Microservices]
↓
[Log Emitters: Fluent Bit / Filebeat / Custom]
↓
[Apache Pulsar Topics (logs)]
↓
[Pulsar IO Elasticsearch Sink Connector]
↓
[Elasticsearch]
↓
[Kibana Dashboards]
```

---

#### Step 1: Ingest Logs into Pulsar

You can emit logs to Pulsar using:
- Fluent Bit with HTTP or Pulsar output plugin
- Filebeat with Kafka → Pulsar bridge
- Custom log appender (e.g., Log4j2 Pulsar Appender)

Example using Pulsar HTTP Producer:

```bash
curl -X POST http://localhost:8080/admin/v3/persistent/public/default/logs \
-H "Content-Type: application/json" \
-d '{"level":"INFO", "msg":"User login successful", "service":"auth-service", "ts":"2024-04-13T10:15:00Z"}'
```

---

#### Step 2: Set Up the Pulsar Elasticsearch Sink Connector

Use **Pulsar IO** to push log messages to Elasticsearch automatically.

Deploy the connector:

```bash
bin/pulsar-admin sink create \
--archive connectors/pulsar-io-elastic-search-<version>.nar \
--sink-config-file es-sink-config.json \
--name elasticsearch-sink \
--inputs logs \
--sink-type elastic_search
```

Sample `es-sink-config.json`:

```json
{
"elasticSearchUrl": "http://elasticsearch:9200",
"indexName": "logs-index",
"typeName": "_doc",
"username": "elastic",
"password": "changeme",
"schemaEnable": false,
"batchSize": 200
}
```

---

#### Step 3: Define Index Mappings in Elasticsearch (Optional)

Define a custom index mapping for structured logs:

```json
PUT /logs-index
{
"mappings": {
"properties": {
"level": { "type": "keyword" },
"msg": { "type": "text" },
"service": { "type": "keyword" },
"ts": { "type": "date" }
}
}
}
```

This ensures optimized search performance and proper timestamp parsing.

---

#### Step 4: Visualize Logs in Kibana

1. Launch **Kibana** and connect to your Elasticsearch instance
2. Create an **index pattern** for `logs-index`
3. Use the **Discover** tab to view live logs
4. Build **visualizations** for error rates, service activity, and log levels

---

#### Real-World Use Cases

- **DevOps Monitoring**: Track application errors, warnings, and slow requests
- **Security Analytics**: Detect login anomalies, DDoS patterns
- **SRE Dashboards**: Real-time latency breakdown and service traces
- **Compliance Auditing**: Long-term retention and searchability of access logs

---

#### Performance and Scaling Tips

- Enable Pulsar **batching** to reduce ingestion overhead
- Use **dedicated Pulsar topics** per service or environment
- Deploy **multiple sink instances** for parallel writes
- Set **index lifecycle policies** in Elasticsearch to manage storage costs
- Tune **heap and indexing buffer** sizes for high-throughput ingestion

---

#### Monitoring and Alerts

Monitor Pulsar and Elasticsearch using:
- **Prometheus + Grafana**
- **Elastic APM**
- **Pulsar Admin CLI**
- Log alerts via **ElastAlert** or **Watcher**

Set alerts for:
- Spike in error logs
- Services with no logs for extended periods
- High indexing delay or search latency

---

#### Conclusion

Apache Pulsar and Elasticsearch offer a highly scalable, real-time solution for log aggregation, search, and visualization. By leveraging **Pulsar IO connectors**, **structured log formats**, and **Elasticsearch analytics**, you can build an end-to-end log observability stack that meets modern DevOps, SRE, and compliance needs.

This architecture scales from single microservices to global multi-tenant environments — giving you full control over your logs, from ingestion to insights.
