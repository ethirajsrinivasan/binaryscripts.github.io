---
layout: post
title: Building a Scalable Messaging Platform with Pulsar and Docker
subtitle: Deploy and scale Apache Pulsar using Docker for modern event-driven systems
categories: Pulsar
tags: [Pulsar, Docker, Messaging, Scalability, Event Streaming, DevOps, Microservices]
excerpt: Learn how to deploy Apache Pulsar with Docker and design a scalable messaging platform. Discover best practices for containerized environments, configuration tuning, and scaling strategies.
---
Apache Pulsar is a cloud-native, distributed messaging platform designed to handle millions of messages per second. With built-in **multi-tenancy**, **geo-replication**, and **decoupled storage/compute architecture**, it is an excellent choice for building modern event-driven systems.

Using **Docker**, you can quickly set up and scale Pulsar clusters for development, testing, and even production environments. In this guide, we’ll walk through building a **scalable messaging platform** with Pulsar and Docker, including architectural components, Docker Compose setup, scaling tips, and best practices.

---

#### Why Use Pulsar with Docker?

Benefits of containerizing Pulsar:

- **Simplified deployment**: Easily run and manage brokers, bookies, and ZooKeeper nodes
- **Isolation**: Avoid conflicts with other services
- **Portability**: Consistent across environments
- **Scalability**: Scale services independently using `docker-compose` or Kubernetes

---

#### Pulsar Core Components

| Component   | Role                                           |
|-------------|------------------------------------------------|
| **Broker**  | Handles producers, consumers, and topic routing|
| **BookKeeper (Bookie)** | Stores message data persistently      |
| **ZooKeeper** | Manages metadata and cluster coordination    |
| **Proxy** (optional) | Routes client traffic in secure clusters |

In Docker, each of these components can be containerized and scaled independently.

---

#### Setting Up Pulsar with Docker Compose

Here's a basic `docker-compose.yml` for a single-node Pulsar setup (for development):

```yaml
version: '3'
services:
zookeeper:
image: apachepulsar/pulsar:3.0.0
command: bin/pulsar zookeeper
container_name: pulsar-zookeeper
ports:
- "2181:2181"

bookie:
image: apachepulsar/pulsar:3.0.0
command: bin/pulsar bookie
container_name: pulsar-bookie
depends_on:
- zookeeper
ports:
- "3181:3181"

broker:
image: apachepulsar/pulsar:3.0.0
command: bin/pulsar broker
container_name: pulsar-broker
depends_on:
- zookeeper
- bookie
ports:
- "6650:6650"   # Pulsar binary protocol
- "8080:8080"   # Pulsar admin REST API
```

Start the cluster:

```bash
docker-compose up -d
```

---

#### Producing and Consuming Messages

Test Pulsar from within the broker container:

```bash
docker exec -it pulsar-broker bin/pulsar-client produce my-topic --messages "Hello from Docker!"
docker exec -it pulsar-broker bin/pulsar-client consume my-topic -s my-sub -n 1
```

This confirms end-to-end functionality.

---

#### Scaling Pulsar Components

To build a **scalable cluster**, you need:

- Multiple **brokers** (horizontally scalable, stateless)
- Multiple **bookies** (scale storage layer)
- A **shared ZooKeeper ensemble** (odd-numbered nodes for quorum)

Example scaling brokers in Docker Compose:

```yaml
broker2:
image: apachepulsar/pulsar:3.0.0
command: bin/pulsar broker
depends_on:
- zookeeper
- bookie
ports:
- "6651:6650"
- "8081:8080"
```

To scale automatically, consider using **Docker Swarm** or **Kubernetes** with Helm charts (`pulsar-helm-chart`).

---

#### Production Configuration Tips

1. **Persistence**: Mount volumes for BookKeeper and ZooKeeper state:

```yaml
volumes:
- ./data/bookie:/pulsar/data/bookkeeper
- ./data/zookeeper:/pulsar/data/zookeeper
  ```

2. **Resource limits**: Set `CPU` and `memory` limits to prevent noisy neighbors.

3. **TLS and Authentication**: Enable security features before exposing brokers externally.

4. **Monitoring**: Expose metrics with Prometheus:

- Use built-in `/metrics` endpoint
- Integrate with Grafana dashboards

---

#### Docker Image Variants

- `apachepulsar/pulsar`: Full-featured, good for development and small clusters
- `apachepulsar/pulsar-all`: Includes tools like `function-worker`, `proxy`, and `dashboard`
- Custom images can be built with minimal footprint for microservice-style deployments

---

#### Example Use Case: Microservices Event Bus

In a microservices architecture, Pulsar with Docker can be used to:

- Connect services using topics
- Handle request/reply over persistent messaging
- Enable retry and DLQ patterns using **dead-letter topics**
- Power real-time analytics with **Pulsar Functions** or **Flink**

---

#### Conclusion

Combining **Apache Pulsar** with **Docker** provides a powerful and flexible way to build **scalable, containerized messaging platforms**. Whether you're running in local dev or preparing for production with Kubernetes, this setup allows for **easy orchestration**, **resilient scaling**, and **high-performance messaging**.

Start simple with Docker Compose, scale out with containers, and let Pulsar power your event-driven applications.
