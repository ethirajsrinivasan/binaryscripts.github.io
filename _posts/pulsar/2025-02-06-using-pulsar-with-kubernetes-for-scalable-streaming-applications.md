---
layout: post
title: Using Pulsar with Kubernetes for Scalable Streaming Applications
subtitle: Deploy Apache Pulsar on Kubernetes to build resilient and scalable streaming architectures
categories: Pulsar
tags: [Apache Pulsar, Kubernetes, Streaming, Cloud Native, Event-Driven, DevOps, Microservices]
excerpt: Learn how to deploy and operate Apache Pulsar on Kubernetes for building scalable, cloud-native streaming applications. Discover architecture, Helm charts, autoscaling, and operational best practices.
---
Apache Pulsar is a cloud-native, distributed messaging and streaming platform designed for **high-performance, multi-tenant** event-driven applications. When paired with **Kubernetes**, Pulsar becomes even more powerful—delivering scalability, resilience, and operational automation out of the box.

In this guide, you'll learn how to use **Pulsar with Kubernetes** to build and deploy scalable streaming applications. We'll explore Pulsar's architecture, how it fits into the Kubernetes ecosystem, and how to leverage tools like Helm, StatefulSets, and autoscalers for robust deployments.

---

#### Why Deploy Pulsar on Kubernetes?

Kubernetes is the de facto standard for container orchestration, offering:

- **Automated deployment and scaling**
- **Built-in high availability and self-healing**
- **Network abstraction for service discovery**
- **Resource isolation for multi-tenancy**

Pulsar complements Kubernetes with:

- **Decoupled architecture** (Brokers, Bookies, ZooKeeper)
- **Multi-tenancy and topic isolation**
- **Geo-replication** and **function runtime support**
- **Tiered storage and fine-grained access control**

Together, they create a platform optimized for **cloud-native streaming at scale**.

---

#### Pulsar Architecture on Kubernetes

Pulsar’s core components:

- **Pulsar Broker** – Handles client connections and message dispatching
- **BookKeeper Bookie** – Manages durable message storage (write-ahead logs)
- **ZooKeeper** – Coordinates metadata and cluster state
- **Proxy (optional)** – Exposes external ingress to clients
- **Functions Worker (optional)** – Executes Pulsar Functions as serverless logic

On Kubernetes, these components are typically deployed using:

- **StatefulSets** for Bookies and ZooKeeper
- **Deployments** for Brokers and Proxies
- **ConfigMaps and Secrets** for configuration management

---

#### Getting Started with Helm Charts

The Apache Pulsar community provides official Helm charts for production-ready deployments.

1. **Add the Pulsar Helm repo**:

```bash
helm repo add apache https://pulsar.apache.org/charts
helm repo update
```

2. **Install with default values** (for testing):

```bash
helm install pulsar apache/pulsar --set initialize=true
```

3. **Customize with a values file** for production:

```yaml
bookkeeper:
replicaCount: 3
broker:
replicaCount: 3
zookeeper:
replicaCount: 3
pulsar:
enableTls: true
tls:
enabled: true
usePulsarCN: true
```

Deploy:

```bash
helm install pulsar -f values.yaml apache/pulsar
```

---

#### Autoscaling and Resource Management

Pulsar on Kubernetes supports **horizontal scaling** of stateless services like:

- Brokers
- Proxies
- Functions Workers

Enable **Horizontal Pod Autoscaler (HPA)** for Brokers:

```bash
kubectl autoscale deployment pulsar-broker \
--cpu-percent=70 --min=3 --max=10
```

Use **resource limits** in Helm config for fine-grained resource allocation.

---

#### Stateful Services: BookKeeper and ZooKeeper

Use **StatefulSets** for Bookies and ZooKeeper to preserve identity and storage volumes.

Best practices:
- Use **PersistentVolumeClaims (PVCs)** with SSD-backed storage
- Anti-affinity rules to spread across zones
- Enable **PodDisruptionBudgets (PDBs)** for graceful rolling updates

---

#### Pulsar Functions and Kubernetes

You can run **Pulsar Functions** (serverless event processing) inside Kubernetes using:

- **K8s Runtime** (built-in integration)
- **Functions Worker** component

Enable the function worker in Helm:

```yaml
function:
enabled: true
runtime: kubernetes
```

Deploy custom Pulsar Functions that run directly in Kubernetes pods, allowing tighter integration with your CI/CD workflows and observability stack.

---

#### Monitoring and Logging

Integrate with Prometheus and Grafana:

- Pulsar exposes **JMX and Prometheus-compatible metrics**
- Use community-provided **Grafana dashboards**
- Monitor:
  - Broker throughput
  - Topic-level lag
  - Bookie disk usage
  - Function execution times

Enable metrics collection via Helm:

```yaml
monitoring:
prometheus:
enabled: true
grafana:
enabled: true
```

---

#### Best Practices

- Use **separate namespaces** for isolation
- Set **network policies** for secure intra-service communication
- Backup and monitor **ZooKeeper state**
- Use **TLS and token authentication** for secure access
- Implement **readiness/liveness probes** for all Pulsar components
- Use **tiered storage** for cost-efficient data retention (e.g., offload to S3)

---

#### Real-World Use Cases

- **IoT Telemetry**: Stream sensor data from devices to cloud in real time
- **Microservices Coordination**: Event bus for service-to-service async communication
- **Edge Analytics**: Process data close to source with Pulsar Functions
- **Streaming ETL**: Ingest → transform → sink with Pulsar IO connectors

---

#### Conclusion

Deploying Apache Pulsar on Kubernetes empowers teams to build **scalable, resilient, and cloud-native streaming applications**. With Helm charts, autoscaling, and native integration into the Kubernetes ecosystem, Pulsar becomes a flexible and powerful platform for event-driven data architectures.

Whether you're modernizing your messaging stack or building greenfield streaming apps, **Pulsar on Kubernetes provides the agility and performance needed to scale with confidence**.
