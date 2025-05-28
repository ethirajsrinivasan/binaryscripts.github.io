---
layout: post
title: Building Highly Available Pulsar Clusters with Cross Data Center Replication
subtitle: Design fault-tolerant Apache Pulsar clusters with built-in geo-replication across data centers
categories: Pulsar
tags: [Pulsar, Apache Pulsar, Geo-Replication, High Availability, Multi-Region, Disaster Recovery, Messaging]
excerpt: Learn how to build highly available Apache Pulsar clusters with cross-data center replication. Explore Pulsar’s geo-replication architecture, setup guides, and best practices for multi-region messaging resilience.
---
As businesses increasingly rely on real-time messaging for critical workflows, ensuring **high availability** and **disaster recovery** becomes essential. Apache Pulsar stands out with **built-in support for geo-replication**, allowing you to easily replicate data across **multiple data centers or cloud regions**.

In this post, we'll explore how to build **highly available Pulsar clusters** using **cross-data center replication (XDR)**. You'll learn about the core architecture, replication mechanisms, configuration steps, and best practices for resilient, multi-region deployments.

---

#### Why Build Multi-Data Center Pulsar Clusters?

Multi-region or cross-data center Pulsar clusters offer:

- **Disaster Recovery (DR)**: Failover capability if a region becomes unavailable
- **Low-Latency Access**: Deliver messages closer to end-users globally
- **High Availability**: Ensure service continuity under network failures or outages
- **Data Residency Compliance**: Keep data within specific geographic boundaries

Pulsar’s native support for **asynchronous geo-replication** makes it easier to deploy such setups compared to other messaging systems.

---

#### Pulsar Geo-Replication Architecture

Pulsar supports **built-in asynchronous replication** across clusters, where:

- Each data center runs an **independent Pulsar cluster**
- Replication is configured at the **namespace level**
- Messages produced in one cluster are **automatically forwarded** to others

```
[Cluster A]   ↔   [Cluster B]
↓                 ↓
[Broker A]       [Broker B]
↓                 ↓
[BookKeeper A]   [BookKeeper B]
```

- Replication is **unidirectional** or **bidirectional**
- Managed by **Replicator Service** running inside brokers

---

#### Setting Up Cross-Cluster Replication

##### 1. Define Clusters

Register both clusters in ZooKeeper:

```bash
bin/pulsar-admin clusters create cluster-a \
--url http://broker-a:8080 \
--broker-url pulsar://broker-a:6650

bin/pulsar-admin clusters create cluster-b \
--url http://broker-b:8080 \
--broker-url pulsar://broker-b:6650
```

##### 2. Create Tenant and Namespace

```bash
bin/pulsar-admin tenants create acme \
--allowed-clusters cluster-a,cluster-b

bin/pulsar-admin namespaces create acme/replicated-ns \
--clusters cluster-a,cluster-b
```

##### 3. Enable Replication

By default, any message published to a replicated topic in one cluster will be replicated to the others:

```
persistent://acme/replicated-ns/my-topic
```

Replication works automatically between the brokers of each region.

---

#### Monitoring Replication Status

Use the CLI to inspect status:

```bash
bin/pulsar-admin topics stats \
persistent://acme/replicated-ns/my-topic
```

Look for:
- `replicationBacklog` – messages yet to replicate
- `connected` – replication channel health
- `msgRateOut` – replication throughput

Also monitor with **Prometheus metrics**:
- `pulsar_replication_backlog`
- `pulsar_replication_rate`

---

#### Failover and Recovery

In a failover scenario:

- Clients can switch from **cluster-a** to **cluster-b**
- All replicated messages are available (with potential delay)
- Subscriptions are maintained if **replicated** (`--replicate-subscriptions`)

Enable subscription sync:

```bash
bin/pulsar-admin namespaces set-subscription-replication acme/replicated-ns --enable
```

This ensures that consumer positions are retained across clusters.

---

#### Best Practices

- Use **dedicated replication clusters** for heavy replication traffic
- **Enable TLS + Auth** on inter-cluster communication
- Ensure **time sync (NTP)** across clusters for log and metric consistency
- Use **Key_Shared subscriptions** for orderly scaling across regions
- Limit replication scope (e.g., only critical topics)
- Back up **ZooKeeper** metadata regularly

---

#### When to Use Active-Active vs. Active-Passive

| Strategy        | Description                                      | Use Case                              |
|----------------|--------------------------------------------------|----------------------------------------|
| Active-Passive | One cluster is primary, other is standby         | DR, cost-conscious setups              |
| Active-Active  | Both clusters actively produce/consume and sync  | Global apps, low-latency requirements  |

Active-active requires **conflict resolution policies** for key collisions and message deduplication.

---

#### Conclusion

Apache Pulsar makes it simple to build **highly available, globally distributed messaging platforms**. With its **native geo-replication**, multi-tenancy, and stateless brokers, Pulsar offers a unique edge in **resilient architecture design**.

By following best practices in **cross-cluster configuration**, **subscription replication**, and **monitoring**, you can ensure that your Pulsar deployment is prepared for **any failure scenario** — and built for **global scale**.
