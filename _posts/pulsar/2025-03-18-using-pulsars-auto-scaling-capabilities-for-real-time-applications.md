---
layout: post
title: Using Pulsar's Auto Scaling Capabilities for Real Time Applications
subtitle: Leverage Apache Pulsar’s dynamic scalability to build elastic and resilient real-time systems
categories: Pulsar
tags: [Pulsar, Auto Scaling, Real-Time, Elasticity, Event Streaming, Apache Pulsar, Scaling, Cloud Native]
excerpt: Discover how Apache Pulsar supports auto-scaling for real-time applications through stateless brokers, topic partitioning, and consumer rebalancing. Build resilient architectures that adapt to demand.
---
Modern real-time applications—from live analytics dashboards to IoT platforms—demand systems that can **scale dynamically** based on data flow. Apache Pulsar is designed with this in mind, offering **built-in auto-scaling capabilities** that adapt to workload changes without downtime.

This blog post explores how to leverage **Pulsar’s auto-scaling architecture** to build **elastic real-time applications**, including strategies for scaling brokers, partitions, and consumers.

---

#### How Pulsar Supports Auto-Scaling

Apache Pulsar achieves elastic scalability through:

- **Stateless Brokers**: Easily add/remove brokers without rebalancing storage
- **Topic Partitioning**: Dynamically scale write/read throughput
- **Shared and Key_Shared Subscriptions**: Enable parallel consumer scaling
- **Decoupled Storage (BookKeeper)**: Allows independent scaling of persistence layer
- **Auto-rebalancing and load shedding**

---

#### Scaling Pulsar Brokers

Pulsar brokers are **stateless** and use **ZooKeeper** to coordinate metadata and topic ownership.

To scale brokers:
- Deploy more broker pods (e.g., in Kubernetes)
- Pulsar will **rebalance topic ownership** across brokers automatically
- Load shedding offloads heavily used topics to less-loaded brokers

**Broker scaling steps:**
1. Increase broker replicas in your orchestration tool
2. Monitor rebalance via Pulsar’s admin API or Prometheus metrics
3. Use **modular load balancer** for fine-grained control

Example Kubernetes deployment scale:

```bash
kubectl scale statefulset pulsar-broker --replicas=6
```

---

#### Scaling Consumers Dynamically

Pulsar’s **Shared** and **Key_Shared** subscription modes support **horizontal scaling** of consumers.

- Add more consumers to a shared subscription to increase throughput
- Messages are distributed across consumers automatically
- No coordination layer required—scaling is instantaneous

**Best practice:** Monitor consumer lag and message backlog to trigger autoscaling.

Auto-scaling consumers with Kubernetes HPA:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
name: pulsar-consumer-hpa
spec:
scaleTargetRef:
apiVersion: apps/v1
kind: Deployment
name: pulsar-consumer
minReplicas: 2
maxReplicas: 10
metrics:
- type: Resource
  resource:
  name: cpu
  target:
  type: Utilization
  averageUtilization: 60
  ```

---

#### Dynamic Topic Partitioning

Partitioned topics in Pulsar allow messages to be distributed across multiple brokers and consumers.

- Each partition is an independent topic under the hood
- Scaling partitions increases parallelism and throughput

Increase partitions using Pulsar Admin:

```bash
bin/pulsar-admin topics update-partitioned-topic \
--partitions 12 persistent://my-tenant/my-ns/my-topic
```

**Note:** Existing consumers must be aware of new partitions, especially in static setups.

---

#### Leveraging Functions and IO Connectors

**Pulsar Functions** and **Pulsar IO connectors** also benefit from Pulsar’s scalability:

- Functions scale across workers using topic partitions
- IO connectors can auto-scale ingestion/export pipelines

Use **parallelism** config to increase function scaling:

```bash
bin/pulsar-admin functions update \
--tenant my-tenant \
--namespace my-ns \
--name enrich-function \
--parallelism 4
```

---

#### Metrics for Auto-Scaling Decisions

Monitor the following to trigger scale-up/down actions:

| Metric                            | Purpose                          |
|----------------------------------|----------------------------------|
| pulsar_subscription_backlog      | Tracks unconsumed messages       |
| pulsar_consumer_throughput       | Indicates consumer saturation    |
| broker_cpu_load                  | Used for broker autoscaling      |
| pulsar_dispatch_rate             | Helps balance topic load         |
| storage_write_latency            | BookKeeper pressure indicator    |

Integrate with Prometheus + Grafana for dashboards and autoscaling triggers.

---

#### Best Practices

- Use **Shared or Key_Shared** subscriptions for parallelism
- Pre-partition topics for high-volume streams
- Use **autoscaling groups** (HPA, ECS, etc.) for consumers and brokers
- Monitor **backlogs** to avoid bottlenecks
- Separate **storage (Bookies)** and **compute (Brokers)** scaling policies
- For cloud-native Pulsar, use **Kubernetes Operators** like StreamNative’s

---

#### Real-World Use Cases

- **IoT Sensor Streams**: Auto-scale consumers as more devices come online
- **E-Commerce Tracking**: Scale brokers and partitions during sales events
- **Log Aggregation**: Increase throughput for ingestion pipelines with volume spikes
- **Real-Time Monitoring**: Dynamically expand processing for alerts and dashboards

---

#### Conclusion

Apache Pulsar’s architecture is purpose-built for **auto-scaling and elasticity**, making it an ideal foundation for modern, real-time applications. By combining **stateless brokers**, **dynamic partitioning**, and **scalable subscriptions**, Pulsar enables systems that adapt to fluctuating loads seamlessly — ensuring consistent performance without over-provisioning.

Whether you're building an IoT platform, streaming ETL, or analytics dashboard, **leveraging Pulsar’s auto-scaling features** will help you deliver reliable, scalable, and cost-efficient event-driven systems.
