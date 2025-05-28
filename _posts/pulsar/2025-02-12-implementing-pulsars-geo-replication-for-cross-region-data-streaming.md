---
layout: post
title: Implementing Pulsar Geo Replication for Cross Region Data Streaming
subtitle: Set up Apache Pulsar's geo-replication to achieve global data distribution with real-time consistency
categories: Pulsar
tags: [Pulsar, Geo-Replication, Cross-Region, Streaming, Real-Time, Multi-DC, Event-Driven, Big Data]
excerpt: Learn how to configure and optimize Apache Pulsar’s built-in geo-replication feature for seamless cross-region streaming and global data consistency.
---
As enterprises expand across geographies, ensuring **real-time, reliable data movement across regions** becomes critical. Apache Pulsar addresses this with **native geo-replication**, enabling seamless streaming of messages between **multiple data centers or cloud regions**.

In this post, we’ll walk through how to **implement Pulsar’s geo-replication**, covering key concepts, setup instructions, best practices, and how it compares to other messaging systems like Kafka.

---

#### What is Geo-Replication in Pulsar?

**Geo-replication** allows Pulsar to **replicate topics across clusters** in different geographical locations. It provides:

- **Cross-region message distribution**
- **Disaster recovery** capabilities
- **Low-latency regional access**
- **Global topic federation** across clusters

Pulsar uses **asynchronous replication** between clusters and supports **multi-active** or **active-passive** patterns.

---

#### Architecture Overview

```
[Producer - Region A]                   [Producer - Region B]
↓                                      ↓
[Pulsar Cluster A] <======> [Pulsar Cluster B] <======> [Pulsar Cluster C]
↓                                      ↓
[Consumer - Region A]                   [Consumer - Region B]
```

Each **Pulsar cluster**:
- Operates independently
- Shares **ZooKeeper metadata**
- Replicates topics using **Replicator service**

---

#### Prerequisites

To enable geo-replication, you need:

1. **Multiple Pulsar clusters** (e.g., `us-west`, `eu-central`)
2. Clusters must be defined in **broker and bookie config**
3. **Shared ZooKeeper / Configuration Store**
4. Proper **network access** and **authentication/authorization** setup between clusters

---

#### Step-by-Step Configuration

##### 1. Define Clusters

Edit `broker.conf` or `standalone.conf` on each cluster:

```properties
clusterName=us-west
```

Use Pulsar CLI to register clusters:

```bash
bin/pulsar-admin clusters create us-west \
--url http://us-west-broker:8080 \
--broker-url pulsar://us-west-broker:6650

bin/pulsar-admin clusters create eu-central \
--url http://eu-central-broker:8080 \
--broker-url pulsar://eu-central-broker:6650
```

---

##### 2. Register Tenants for Replication

Enable tenants across clusters:

```bash
bin/pulsar-admin tenants update my-tenant \
--allowed-clusters us-west,eu-central
```

---

##### 3. Create Topics with Replication Enabled

Enable replication while creating topics:

```bash
bin/pulsar-admin topics create persistent://my-tenant/my-ns/my-topic \
--clusters us-west,eu-central
```

Or configure default replication for a namespace:

```bash
bin/pulsar-admin namespaces set-clusters my-tenant/my-ns \
--clusters us-west,eu-central
```

---

##### 4. Start Replication

Messages published in one region will be **asynchronously propagated** to other clusters.

No changes are needed for producer/consumer logic — Pulsar handles replication automatically behind the scenes.

---

#### Monitoring and Verification

Verify replication status:

```bash
bin/pulsar-admin topics stats persistent://my-tenant/my-ns/my-topic
```

Look for:
- `replicationBacklog`
- `connected` flags for each remote cluster
- Throughput metrics per region

Use **Prometheus + Grafana** dashboards for visual insights.

---

#### Best Practices

✅ Keep cluster names unique and consistent  
✅ Monitor **replication lag** using topic stats  
✅ Use **asynchronous retry policies** for temporary outages  
✅ Use **TLS and authentication** across inter-cluster links  
✅ Avoid circular replication (i.e., A → B → A)  
✅ For DR, replicate critical topics only, not all

---

#### Use Cases

- **Multi-region microservices**: Ensure each region sees all events
- **Cross-cloud hybrid architectures**
- **Disaster Recovery**: Failover to another cluster during outages
- **Analytics**: Send operational data to centralized region for batch or BI

---

#### Pulsar vs Kafka for Replication

| Feature             | Pulsar                          | Kafka                          |
|---------------------|----------------------------------|--------------------------------|
| Native Replication  | ✅ Yes                          | ❌ Requires MirrorMaker        |
| Multi-Tenancy       | ✅ Built-in                      | ❌ External effort             |
| Setup Complexity    | Medium                          | High with MirrorMaker 2        |
| Replication Pattern | Asynchronous + multi-active     | Mostly active-passive          |
| Geo Routing         | Supported via topic policies    | Manual setup needed            |

---

#### Conclusion

Apache Pulsar’s built-in **geo-replication** makes it easy to build **resilient, globally distributed streaming systems**. Whether you're supporting **multi-region consumers**, **low-latency access**, or **disaster recovery**, Pulsar's cross-cluster replication enables you to scale your messaging infrastructure beyond a single data center — with **minimal operational complexity**.

By following these practices and architectural patterns, you can ensure **reliable, scalable, and secure cross-region data streaming** using Apache Pulsar.
