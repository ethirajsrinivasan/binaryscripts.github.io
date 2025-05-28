---
layout: post
title: Configuring Pulsar for Low Latency and High Throughput Use Cases
subtitle: Optimize Apache Pulsar for real-time applications with performance-tuned configuration and deployment strategies
categories: Pulsar
tags: [Pulsar, Low Latency, High Throughput, Messaging, Performance Tuning, Real-Time Systems, PubSub]
excerpt: Learn how to configure Apache Pulsar for low-latency and high-throughput use cases. Explore broker tuning, batching, partitioning, and consumer strategies for building responsive real-time systems.
---
Apache Pulsar is designed for **scalability and performance**, supporting messaging patterns with **low latency** and **high throughput** out of the box. But to truly harness Pulsar’s capabilities for real-time systems — such as **financial applications**, **IoT telemetry**, or **event-driven microservices** — you need to **fine-tune your configuration** and **optimize resource usage** across producers, brokers, and consumers.

This guide walks through the key configuration parameters, deployment best practices, and real-world strategies for building **high-performance Pulsar-based systems**.

---

#### Key Concepts for Tuning

To achieve optimal performance, you need to balance:

| Objective        | Key Factors                                     |
|------------------|-------------------------------------------------|
| Low Latency      | Small batch sizes, quick acknowledgments, dedicated topics |
| High Throughput  | Batching, partitioning, parallel consumption, optimized IO |

Let’s dive into how to tune each part of Pulsar’s pipeline.

---

#### 1. Producer Optimization

**Batching** and **asynchronous writes** help boost throughput.

```java
Producer<byte[]> producer = client.newProducer()
.topic("persistent://public/default/metrics")
.enableBatching(true)
.batchingMaxMessages(1000)
.batchingMaxPublishDelay(10, TimeUnit.MILLISECONDS)
.sendTimeout(0, TimeUnit.SECONDS) // disable timeouts
.compressionType(CompressionType.LZ4)
.create();
```

Best practices:
- Use **compression** (LZ4/Snappy) for throughput-sensitive use cases
- Set `batchingMaxPublishDelay` low (5–10 ms) for faster publishing
- Disable `sendTimeout` to reduce retransmit risk on latency spikes
- Use **partitioned topics** for load balancing across brokers

---

#### 2. Broker and BookKeeper Configuration

Broker tuning is essential for predictable low-latency performance.

Key broker settings in `broker.conf`:

```properties
managedLedgerDefaultEnsembleSize=3
managedLedgerDefaultWriteQuorum=3
managedLedgerDefaultAckQuorum=2

nettyMaxFrameSizeBytes=104857600  # 100MB
maxMessageSize=5242880            # 5MB
```

Performance tips:
- Ensure **dedicated disks for BookKeeper** (ideally SSD)
- Isolate **broker and Bookie workloads** if possible
- Monitor **write latency** via Prometheus metrics (`pulsar_bookie_journal_latency`)

---

#### 3. Consumer Tuning

Use the right **subscription type** and **batching strategy**.

**Shared or Key_Shared** subscriptions allow **parallel processing**:

```java
Consumer<byte[]> consumer = client.newConsumer()
.topic("transactions")
.subscriptionName("transaction-processor")
.subscriptionType(SubscriptionType.Key_Shared)
.receiverQueueSize(1000)
.ackTimeout(60, TimeUnit.SECONDS)
.subscribe();
```

Best practices:
- Increase `receiverQueueSize` for higher throughput
- Use **Key_Shared** for ordering + concurrency
- Use **batch receive** in high-volume pipelines

---

#### 4. Partitioning for Parallelism

Partitioned topics are critical for scaling:

```bash
pulsar-admin topics create-partitioned-topic \
persistent://public/default/events --partitions 12
```

- Producers will hash messages across partitions
- Consumers can process partitions in parallel
- Broker load spreads evenly

Choose number of partitions based on expected message volume and consumer parallelism.

---

#### 5. Thread Pool and Resource Management

In `broker.conf`, adjust worker threads:

```properties
numIOThreads=16
numHttpServerThreads=8
numWorkerThreads=32
```

Tune based on:
- Number of producers/consumers
- Available CPU cores
- Message rate (TPS)

Use tools like **vCPU pinning** and **dedicated memory settings** in containerized environments (e.g., Kubernetes with QoS classes).

---

#### 6. Monitoring and Metrics

Track real-time performance with:
- **Prometheus + Grafana dashboards**
- Key metrics:
  - `pulsar_in_bytes_total`
  - `pulsar_out_bytes_total`
  - `pulsar_publish_latency`
  - `pulsar_dispatch_rate`

Set **alerts** for:
- Increased message backlog
- High broker memory usage
- Message processing delays

---

#### 7. Network and Storage Considerations

- Use **low-latency networking** (10G or more for high throughput)
- Separate **journal and ledger disks** on Bookies
- Use **NVMe or SSDs** for BookKeeper write-ahead logs
- Enable **async disk flushes** only if data loss is tolerable

---

#### 8. Advanced Features

Enable **DLQ (Dead Letter Queues)** to prevent stuck consumers:

```
maxRedeliverCount=5
deadLetterPolicy=DeadLetterPolicy.builder()
.deadLetterTopic("persistent://public/default/dlq")
.build();
```

Use **message deduplication** for idempotency and throughput tuning.

---

#### Conclusion

To unlock Apache Pulsar’s full potential for **low-latency** and **high-throughput messaging**, you must tune each layer — producers, brokers, BookKeeper, and consumers — with workload-specific parameters. By batching efficiently, parallelizing with partitioned topics, and monitoring performance, you can build messaging pipelines that meet the most demanding SLAs in real-time environments.

Whether you're streaming sensor data, processing financial transactions, or powering real-time analytics, Pulsar can deliver — when configured right.
