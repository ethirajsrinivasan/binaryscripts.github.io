---
layout: post
title: Implementing Pulsar for Event Sourcing and CQRS Architectures
subtitle: Build scalable, event-driven microservices using Apache Pulsar for event sourcing and CQRS patterns
categories: Pulsar
tags: [Pulsar, Event Sourcing, CQRS, Microservices, Event-Driven, Architecture, Apache Pulsar, Domain Driven Design]
excerpt: Learn how to implement event sourcing and Command Query Responsibility Segregation (CQRS) using Apache Pulsar. Explore topic design, persistence strategies, and consumer patterns for building scalable microservices.
---
As distributed systems and microservices evolve, architectural patterns like **Event Sourcing** and **CQRS (Command Query Responsibility Segregation)** are becoming essential for scalability, auditability, and flexibility. **Apache Pulsar** is a powerful event-streaming platform that offers exactly the features needed to implement these patterns effectively — including **durable logs**, **multi-subscription models**, and **low-latency message delivery**.

In this post, we’ll explore how to implement **Event Sourcing and CQRS architectures using Apache Pulsar**, covering concepts, patterns, and real-world design strategies.

---

#### What is Event Sourcing?

**Event Sourcing** is a technique where state changes in an application are stored as a sequence of **immutable events** rather than just updating the current state in a database.

Instead of:

```
UPDATE account SET balance = balance - 100 WHERE id = 1;
```

You emit:

```
Event: MoneyWithdrawn(accountId=1, amount=100, timestamp=...)
```

The current state is **reconstructed by replaying** all the past events in order.

---

#### What is CQRS?

**CQRS** separates the responsibilities of:
- **Commands** — requests that change state (e.g., CreateOrder)
- **Queries** — requests that read state (e.g., GetOrderDetails)

This separation allows you to:
- Scale reads and writes independently
- Use different data models for reading vs writing
- Optimize performance, availability, and complexity

When combined with **event sourcing**, commands generate events, and queries subscribe to those events to build materialized views.

---

#### Why Apache Pulsar?

Apache Pulsar is well-suited for event-driven systems due to:

- **Durable storage** with persistent logs
- **Message replay** for state recovery
- **Multiple subscription modes** (exclusive, shared, failover)
- **Message ordering and delivery guarantees**
- **Built-in topic partitioning and multi-tenancy**

Pulsar allows **event sourcing and CQRS patterns to scale cleanly** in a distributed microservices setup.

---

#### Architectural Overview

```
+-------------+           +-------------+
|  Command    |           |  Query      |
|  Service    |           |  Service    |
+------+------+           +------+------+
|                         ^
|                         |
v                         |
+---------------+        +---------------+
| Pulsar Topic  |------->|  Read Model DB |
| (Event Stream)|        +---------------+
+---------------+
|
v
+--------------------+
|  Event Store / Log |
+--------------------+
```

- Commands produce events to Pulsar
- Event consumers update read models (views)
- Queries access optimized, read-only materialized views

---

#### Implementing Event Sourcing with Pulsar

Each **aggregate (e.g., order, user)** can publish domain events to its own topic or a shared topic.

Example event (JSON):

```json
{
"eventType": "OrderPlaced",
"orderId": "ORD123",
"customerId": "CUS456",
"timestamp": "2024-11-16T10:00:00Z",
"items": [{"sku": "ABC", "qty": 2}]
}
```

You can publish this via Pulsar's client SDKs:

```python
from pulsar import Client

client = Client('pulsar://localhost:6650')
producer = client.create_producer('persistent://orders/events')

producer.send(json.dumps(event).encode('utf-8'))
client.close()
```

---

#### Implementing CQRS with Pulsar

For **query-side** services:

- Subscribe to event topics
- Consume and apply events to build denormalized views (e.g., PostgreSQL, Redis, Elasticsearch)

Consumer example in Python:

```python
consumer = client.subscribe('persistent://orders/events', subscription_name='view-builder')

while True:
msg = consumer.receive()
event = json.loads(msg.data())

    if event['eventType'] == 'OrderPlaced':
        # Update view database
        update_orders_view(event)

    consumer.acknowledge(msg)
```

Use **failover or shared subscription** depending on throughput and durability needs.

---

#### Topic and Event Design

Best practices:
- Use **namespaced topics per domain**: `persistent://tenant/orders/events`
- Use **event versioning** to evolve schemas safely
- Use **keyed messages** for partitioning by aggregate (e.g., order ID)
- Store events in **Avro/JSON** with schema registry integration for validation

---

#### Handling Replay and Recovery

Pulsar supports **message replay** by:
- Resetting subscription cursors
- Using message IDs or publish timestamps

```bash
pulsar-admin topics reset-cursor --subscription view-builder \
--to-timestamp 1699990000000 persistent://orders/events
```

This allows rebuilding read models or correcting logic after bugs.

---

#### Ensuring Consistency and Ordering

- Use **single partition per aggregate** to ensure event order
- Use **transactional outbox pattern** for ensuring DB + Pulsar sync
- Implement **idempotent consumers** to avoid duplicate processing
- Use **Pulsar Functions** to transform/route events inline if needed

---

#### Monitoring and Observability

Monitor:
- **Consumer lag**
- **Unacknowledged messages**
- **Backlog growth**
- **Replay volume**

Use tools like:
- Pulsar Manager
- Prometheus + Grafana
- StreamNative Console

---

#### Best Practices

✅ Design immutable, append-only events  
✅ Use explicit event types and versioning  
✅ Apply schema validation for all published messages  
✅ Use read models optimized for query patterns  
✅ Automate subscription reset and replay workflows  
✅ Consider dead-letter topics for invalid/unprocessable messages

---

#### Conclusion

Apache Pulsar provides all the building blocks needed for robust, scalable **event sourcing and CQRS architectures**. With its powerful messaging model, persistent logs, and subscription flexibility, Pulsar empowers teams to build **audit-friendly**, **resilient**, and **event-driven microservices** with full control over message flow and state evolution.

Whether you're modernizing a legacy monolith or designing a new system, Pulsar can anchor your **domain-driven**, real-time architecture.
