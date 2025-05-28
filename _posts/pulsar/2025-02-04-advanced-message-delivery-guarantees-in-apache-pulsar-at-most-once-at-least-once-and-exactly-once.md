---
layout: post
title: Advanced Message Delivery Guarantees in Apache Pulsar At Most Once At Least Once and Exactly Once
subtitle: Understand how Apache Pulsar delivers messages with precision using configurable delivery semantics
categories: Pulsar
tags: [Pulsar, Message Delivery, At-least-once, Exactly-once, PubSub, Messaging, Streaming Systems]
excerpt: Explore Apache Pulsar’s support for at-most-once, at-least-once, and exactly-once delivery guarantees. Learn how these semantics are implemented, configured, and used in real-world messaging applications.
---
Modern messaging systems must ensure that messages are delivered **reliably and consistently** to consumers. Apache Pulsar provides **flexible message delivery guarantees** — including **at-most-once**, **at-least-once**, and **exactly-once** — giving developers the tools to balance **latency**, **throughput**, and **consistency**.

In this blog, we’ll explore how Apache Pulsar implements these delivery semantics, how to configure them, and when to use each guarantee for optimal results.

---

#### What Are Delivery Guarantees?

| Delivery Type   | Description                                                                 |
|------------------|------------------------------------------------------------------------------|
| At-most-once     | Messages may be lost but are never redelivered                              |
| At-least-once    | Messages are never lost but may be delivered more than once                 |
| Exactly-once     | Each message is delivered once and only once                                |

Each guarantee serves different use cases — from low-latency, lossy applications to financial-grade, deduplicated pipelines.

---

#### Pulsar Architecture Recap

Pulsar decouples **compute (brokers)** and **storage (BookKeeper)**. It supports multiple **subscription modes**, **acknowledgement mechanisms**, and **message deduplication**, which together enable fine-grained delivery guarantees.

---

#### At-Most-Once Delivery in Pulsar

**At-most-once** means a message is delivered **zero or one time** — there is no redelivery if the consumer fails before acknowledgment.

How it works in Pulsar:
- Consumer **acknowledges messages immediately** upon receipt
- No retry or redelivery is attempted
- Ideal for **non-critical telemetry**, metrics, or best-effort logging

Enable using:
- Manual ack strategy with immediate `acknowledgeAsync()`
- No retry logic in consumer code

**Example:**

```java
consumer.receiveAsync().thenAccept(msg -> {
// process
consumer.acknowledgeAsync(msg); // ack immediately, no retry
});
```

---

#### At-Least-Once Delivery in Pulsar

This is the **default delivery guarantee** in Pulsar. Messages will be delivered **at least once**, meaning duplicates are possible in case of processing failure.

How it works:
- Broker resends unacknowledged messages
- Messages are retained until explicitly acknowledged
- Durable, safe for **critical data ingestion**, **transactional processing**, etc.

Enable using:
- Subscription type: `Shared`, `Exclusive`, or `Failover`
- Acknowledgement required from consumer

**Example:**

```java
Message<byte[]> msg = consumer.receive();
try {
process(msg);
consumer.acknowledge(msg); // only after successful processing
} catch (Exception e) {
consumer.negativeAcknowledge(msg); // triggers redelivery
}
```

**Use Case Examples:**
- Payment processing
- Order ingestion
- Inventory updates

---

#### Exactly-Once Delivery in Pulsar

**Exactly-once** ensures that a message is **delivered and processed only once**, even across retries and broker failures.

Pulsar supports this via:
- **Message deduplication** (producer-side or broker-side)
- **Transactions** (introduced in Pulsar 2.8+)

##### Producer Deduplication

Enabled per topic:

```properties
producerName=my-producer
enableDeduplication=true
```

Each message includes a unique ID (`sequenceId`), and the broker stores deduplication metadata.

##### Pulsar Transactions

A transaction allows:
- Publishing messages to multiple topics **atomically**
- Acknowledging messages **within a transaction**
- Committing/aborting based on application logic

**Example:**

```java
Transaction txn = pulsarClient.newTransaction()
.withTransactionTimeout(5, TimeUnit.MINUTES)
.build().get();

consumer.acknowledgeAsync(message, txn);
producer.newMessage(txn).value(data).sendAsync();

txn.commit().get();
```

**Use Cases:**
- Banking, trading platforms
- Event sourcing
- Distributed ledgers

---

#### Delivery Guarantees by Subscription Type

| Subscription Type | At-most-once | At-least-once | Exactly-once |
|-------------------|--------------|----------------|---------------|
| Exclusive         | ✅            | ✅              | ✅ (with TXNs) |
| Shared            | ✅            | ✅              | ✅ (dedupe only) |
| Failover          | ✅            | ✅              | ✅ (with TXNs) |
| Key_Shared        | ✅            | ✅              | ✅ (ordered, dedupe) |

---

#### Monitoring and Tuning

- **Enable message deduplication** on brokers for global support
- Monitor **acknowledgment latency**, **redelivery count**, and **transactional errors**
- Use **dead-letter topics (DLQ)** for poison message handling
- Set **ack timeout** (`ackTimeoutMillis`) to tune redelivery behavior

---

#### Best Practices

✅ Use **at-least-once** as the default unless loss is acceptable  
✅ Enable **producer name and sequenceId** for deduplication  
✅ Use **transactions** for strong exactly-once delivery guarantees  
✅ Apply **idempotent processing** logic on consumer side  
✅ Monitor DLQ and retry metrics to avoid infinite reprocessing  
✅ Segment topics and use **Key_Shared** for parallel processing with ordering

---

#### Conclusion

Apache Pulsar offers **fine-grained control over message delivery semantics**, allowing teams to build applications that are resilient, scalable, and tailored to their consistency needs. Whether you need **high-throughput at-most-once ingestion** or **financial-grade exactly-once workflows**, Pulsar provides the tools to achieve your goals — with a modern, cloud-native messaging foundation.
