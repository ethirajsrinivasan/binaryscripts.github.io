---
layout: post
title: Understanding Apache Pulsar Architecture Brokers Topics and Storage
subtitle: Explore the internal architecture of Apache Pulsar including brokers, topics, and the decoupled storage model
categories: Pulsar
tags: [Pulsar, Apache Pulsar, Messaging, PubSub, Architecture, Distributed Systems, Event Streaming]
excerpt: Gain deep insights into Apache Pulsar's architecture by understanding the roles of brokers, topics, BookKeeper, and the separation of compute and storage for scalable messaging systems.
---
**Apache Pulsar** is a next-generation distributed messaging and streaming platform that offers **high throughput**, **low latency**, and **multi-tenancy** out of the box. What sets Pulsar apart from traditional messaging systems like Apache Kafka is its **decoupled architecture**, where compute (brokers) and storage (BookKeeper) are separated.

In this post, we’ll explore the core components of **Apache Pulsar’s architecture** — including **brokers, topics, namespaces, and storage layers** — and how they work together to provide a scalable, resilient, and flexible messaging system.

---

#### Pulsar Architectural Overview

Apache Pulsar is made up of three primary components:

1. **Brokers** – Route messages and manage client connections
2. **Topics** – Logical streams for publishing/subscribing
3. **BookKeeper** – Persistent storage layer for messages

This design enables **elastic scalability**, **high availability**, and **geographic distribution**.

---

#### Pulsar Broker

The **broker** is the stateless component of Pulsar that:

- Accepts connections from producers and consumers
- Handles subscription management
- Authenticates and authorizes client requests
- Routes messages to the appropriate storage layer (BookKeeper)
- Loads topic metadata from **ZooKeeper** and **Topic Ownership Service**

Because brokers are stateless, they can be **horizontally scaled** without data loss or migration complexity.

---

#### Topics in Pulsar

Pulsar organizes data into **topics**, which are:

- Logical append-only logs
- Partitioned for parallelism (optional)
- Hosted within **namespaces**, which are grouped under **tenants**

Topic naming convention:

```
persistent://<tenant>/<namespace>/<topic>
```

Example:

```
persistent://acme/iot/temperature
```

There are two types of topics:

- **Persistent**: Stored durably using BookKeeper
- **Non-persistent**: Stored in memory, best-effort delivery

Each partition of a topic is independently distributed and replicated across brokers.

---

#### Message Storage with Apache BookKeeper

Unlike Kafka, which stores data on the broker’s local disk, Pulsar uses **Apache BookKeeper** as a dedicated **storage layer**.

How it works:

- Messages are written to a **ledger**, which is a sequence of entries stored in BookKeeper
- Ledgers are replicated across **bookies** for durability
- Once a ledger is full, a new ledger is created, and the metadata is updated

Benefits of this model:

- **True separation of compute and storage**
- **Better fault tolerance** (data survives broker restarts)
- Enables **tiered storage** (offload to S3, GCS, etc.)

---

#### Pulsar Namespaces and Tenants

To support **multi-tenancy**, Pulsar uses a hierarchical resource model:

- **Tenant**: A logical grouping (e.g., a company or app)
- **Namespace**: A resource space for topics, policies, and quotas
- **Topics**: The actual message streams

Admins can apply **quotas, authentication rules, and retention policies** at the namespace level, enabling isolated and secure usage per tenant.

---

#### Message Retention and Acknowledgments

Pulsar supports advanced retention strategies:

- **Message retention**: Keep acknowledged messages for delayed consumers
- **Acknowledgment levels**:
  - Individual ack
  - Cumulative ack (efficient for ordered consumption)

Combined with **subscriptions**, this allows flexible delivery models:

- **Exclusive**
- **Shared (Round Robin)**
- **Failover**
- **Key_Shared** (order-preserving for keys)

---

#### Geo-Replication

Pulsar supports **built-in geo-replication** between clusters using **Pulsar Replication Service**.

Advantages:
- No external tools needed
- Simple configuration at namespace level
- Active-active or active-passive patterns

Example config:

```
bin/pulsar-admin namespaces set-clusters \
--clusters us-west,eu-central my-tenant/my-namespace
```

---

#### Scaling and Fault Tolerance

- **Brokers** scale independently from **bookies**
- **BookKeeper** handles replication and durability
- Brokers use **ZooKeeper** for service discovery and metadata management
- Failed brokers do not lose data — leadership transfers to another broker

This model ensures **resilient message delivery** and **elastic scaling** across compute and storage layers.

---

#### Key Benefits of Pulsar's Architecture

| Feature                     | Description                                           |
|-----------------------------|-------------------------------------------------------|
| Decoupled Compute/Storage   | Independent scaling of brokers and bookies           |
| Stateless Brokers           | Simplifies elasticity and failover                   |
| Multi-Tenancy               | Namespace and tenant-level isolation                 |
| Durability via BookKeeper   | High-availability storage layer                      |
| Built-in Geo-Replication    | Simplified DR and multi-region deployments           |
| Flexible Subscriptions      | Multiple consumer models for different use cases     |
| Tiered Storage              | Offload old data to cloud storage (S3, GCS)          |

---

#### Conclusion

Apache Pulsar’s architecture was designed from the ground up to support **cloud-native**, **multi-tenant**, and **real-time messaging applications**. By separating brokers and storage, Pulsar enables better scalability, resilience, and operational simplicity.

Whether you're building **IoT platforms**, **financial data pipelines**, or **enterprise messaging systems**, understanding Pulsar's core architecture is the first step toward designing high-performance and future-proof streaming applications.
