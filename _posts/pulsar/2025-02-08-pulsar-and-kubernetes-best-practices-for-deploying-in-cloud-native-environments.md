---
layout: post
title: Pulsar and Kubernetes Best Practices for Cloud Native Deployments
subtitle: A technical guide to deploying Apache Pulsar on Kubernetes with Helm, TLS, autoscaling and observability
categories: Pulsar
tags: [Apache Pulsar, Kubernetes, Helm, Cloud Native, Big Data, Event Streaming, Messaging, DevOps, HDFS, Spark]
excerpt: Learn how to deploy Apache Pulsar on Kubernetes using Helm charts, persistent storage, authentication, autoscaling, and monitoring. This guide covers best practices for cloud-native deployments optimized for performance, security, and scalability.
---
Apache Pulsar is a modern, cloud-native messaging and event streaming platform built for scalability, resilience, and multi-tenancy. When paired with Kubernetes, Pulsar becomes a powerful engine for real-time data pipelines, supporting everything from microservices communication to big data ingestion.

This post is targeted at **intermediate to advanced users** and explores *technical best practices* for deploying **Pulsar on Kubernetes** using **Helm**, with a focus on security, autoscaling, observability, and resource isolation. Whether you're running a single-tenant SaaS platform or a multi-region event mesh, these strategies will help you build a robust production-grade deployment.

---

#### Why Deploy Pulsar on Kubernetes

Apache Pulsar was designed with a **cloud-native mindset**, making Kubernetes an ideal environment for hosting it. Benefits include:

- **Pod autoscaling and resilience**
- **Helm-based templating for configuration management**
- **Persistent volume management via CSI drivers**
- **Namespace and RBAC isolation for multi-tenancy**
- **Seamless rolling updates and TLS certificate rotation**

By containerizing Pulsar components like Brokers, Bookies (BookKeeper), ZooKeeper, and Proxies, you can achieve full control over **infrastructure, fault tolerance, and security**.

---

#### Helm Chart Installation Overview

The Apache Pulsar project provides an official Helm chart that simplifies deployment.

```bash
helm repo add apache https://pulsar.apache.org/charts
helm repo update
helm install pulsar apache/pulsar --namespace pulsar --create-namespace -f values.yaml
```

The chart supports granular configuration of each component via `values.yaml`. You can configure TLS, autoscaling, authentication, service types, and persistence volumes using this file.

---

#### Sample Helm Configuration for Production Use

Here is a simplified yet production-ready `values.yaml` configuration:

```yaml
zookeeper:
replicas: 3
persistence:
enabled: true
size: 50Gi
storageClass: "fast-disks"

bookkeeper:
replicas: 3
persistence:
enabled: true
size: 100Gi
storageClass: "fast-disks"

broker:
replicas: 3
service:
type: LoadBalancer
configData:
PULSAR_PREFIX_authenticationEnabled: "true"
PULSAR_PREFIX_authorizationEnabled: "true"
PULSAR_PREFIX_authenticationProviders: "org.apache.pulsar.broker.authentication.AuthenticationProviderToken"
PULSAR_PREFIX_tokenSecretKey: "file:///pulsar/token/secret.key"

proxy:
enabled: true
replicas: 2
service:
type: LoadBalancer

tls:
enabled: true
broker:
enabled: true
proxy:
enabled: true

grafana:
enabled: true

prometheus:
enabled: true

pulsar_manager:
enabled: true

autoscaling:
broker:
enabled: true
minReplicas: 2
maxReplicas: 10
```

---

#### TLS and Authentication

Security is a non-negotiable aspect of modern messaging systems. Pulsar supports **TLS encryption** and **JWT/OAuth2-based authentication** out of the box.

**Enable TLS for Brokers and Proxies:**
- Create Kubernetes secrets for your certificates
- Reference them in the Helm chart under `tls.broker.enabled` and `tls.proxy.enabled`

**Enable Authentication and Authorization in Broker Config:**

```yaml
broker:
configData:
PULSAR_PREFIX_authenticationEnabled: "true"
PULSAR_PREFIX_authorizationEnabled: "true"
PULSAR_PREFIX_authenticationProviders: "org.apache.pulsar.broker.authentication.AuthenticationProviderToken"
PULSAR_PREFIX_tokenSecretKey: "file:///pulsar/token/secret.key"
```

Use CLI to assign roles:

```bash
bin/pulsar-admin namespaces grant-permission tenantA/namespace1 \
--actions produce,consume \
--role token-user
```

---

#### Observability and Metrics

**Prometheus and Grafana** integration is supported by default in the Helm chart. Enable them in `values.yaml`:

```yaml
prometheus:
enabled: true

grafana:
enabled: true
```

Monitor key metrics such as:
- `pulsar_publish_rate`
- `pulsar_throughput_in`
- `bookkeeper_write_latency`
- `zookeeper_sessions`

Set up **Grafana dashboards** for per-topic and per-tenant monitoring, and use **Pulsar Manager UI** for topic-level insights and cluster health.

---

#### Scaling and Resource Optimization

Apache Pulsar can be scaled horizontally by increasing replicas of Brokers and Proxies. For large clusters:

- Use **Horizontal Pod Autoscaler (HPA)** for Brokers and Proxy Pods
- Use **node affinity and taints** to isolate BookKeeper on storage-optimized nodes
- Separate **ZooKeeper, BookKeeper, and Broker workloads** into different node pools
- Enable **tiered storage** using cloud buckets to offload old data

You can also tune BookKeeper with different journal and ledger disk configurations for optimal write performance.

---

#### Kubernetes Best Practices

**Namespace Isolation**  
Deploy Pulsar in its own namespace to isolate it from other workloads.

**Pod Affinity and Anti-Affinity**  
Ensure BookKeeper and Broker pods do not co-locate using anti-affinity rules.

**StatefulSet Deployment**  
Deploy ZooKeeper and BookKeeper as StatefulSets with Persistent Volume Claims for fault tolerance.

**Readiness and Liveness Probes**  
Use Pulsar-provided health endpoints to implement custom probes and ensure rolling upgrades do not affect availability.

---

#### Advanced Features for Enterprise Use

- **Multi-Cluster Setup**: Use Pulsar’s geo-replication to connect clusters across regions.
- **Tiered Storage**: Offload old segments to S3 or GCS.
- **Functions and IO Connectors**: Use Pulsar Functions to do stream processing without an external processing engine.
- **Pulsar Proxy**: Deploy proxies to act as secure ingress controllers for client traffic.

---

#### Conclusion

Apache Pulsar and Kubernetes form a **highly scalable and secure combination** for building modern, cloud-native streaming platforms. By following these best practices—secure Helm deployments, observability, autoscaling, and resource isolation—you can confidently run Pulsar in production across a variety of use cases.

Whether you're ingesting billions of IoT events or building a real-time analytics pipeline, **Pulsar on Kubernetes is a future-ready choice** for resilient and manageable data streaming.

