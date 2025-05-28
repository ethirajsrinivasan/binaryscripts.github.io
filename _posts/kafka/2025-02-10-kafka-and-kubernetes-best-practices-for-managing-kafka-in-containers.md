---
layout: post
title: Kafka and Kubernetes Best Practices for Managing Kafka in Containers
subtitle: Learn how to run and manage Apache Kafka effectively on Kubernetes for scalable and resilient event streaming
categories: Kafka
tags: [Kafka, Kubernetes, Event Streaming, Containers, DevOps, Cloud-Native, Apache Kafka, K8s]
excerpt: Discover best practices for deploying and managing Apache Kafka on Kubernetes. Learn how to handle stateful workloads, ensure high availability, optimize storage, and use operators effectively for resilient event streaming.
---
Apache Kafka has become the backbone of modern **event-driven architectures**, enabling real-time data pipelines and microservices. As organizations move toward **cloud-native infrastructure**, running Kafka on **Kubernetes** offers flexibility, scalability, and ease of automation — but it also introduces new challenges.

Kafka is a **stateful, distributed system**, which means managing it inside a containerized environment like Kubernetes requires careful planning.

In this blog, we explore **best practices for running Apache Kafka on Kubernetes**, including deployment strategies, storage configuration, security, and operational tooling.

---

#### Why Run Kafka on Kubernetes?

- **Cloud-native orchestration** with self-healing and scaling
- **Standardized deployments** across environments
- **Better resource isolation** with namespaces and quotas
- **DevOps automation** with Helm, Operators, and CI/CD
- **Service discovery** and **load balancing** built in

Despite these advantages, Kafka’s dependencies on **persistent storage**, **network identity**, and **partition assignment** require special attention in Kubernetes.

---

#### 1. Use StatefulSets for Kafka Brokers

Kafka brokers should be deployed using **StatefulSets**, not Deployments, because they:

- Provide **stable network identities** (e.g., `kafka-0`, `kafka-1`)
- Enable **persistent storage** using PVCs (PersistentVolumeClaims)
- Maintain **order during scaling and rolling updates**

Example StatefulSet configuration:

```yaml
kind: StatefulSet
metadata:
name: kafka
spec:
serviceName: "kafka"
replicas: 3
selector:
matchLabels:
app: kafka
template:
metadata:
labels:
app: kafka
spec:
containers:
- name: kafka
image: confluentinc/cp-kafka:7.3.0
ports:
- containerPort: 9092
volumeMounts:
- name: data
mountPath: /var/lib/kafka/data
volumeClaimTemplates:
- metadata:
name: data
spec:
accessModes: ["ReadWriteOnce"]
storageClassName: gp2
resources:
requests:
storage: 100Gi
```

---

#### 2. Use Headless Services for Broker Discovery

Kafka clients and brokers need to communicate directly. Use **headless services** to expose StatefulSet pods with consistent DNS.

```yaml
apiVersion: v1
kind: Service
metadata:
name: kafka
spec:
clusterIP: None
selector:
app: kafka
ports:
- port: 9092
```

Each broker will be reachable at:  
`kafka-0.kafka.default.svc.cluster.local`

---

#### 3. Use Kafka Operators for Automation

Operators like **Strimzi**, **Confluent Operator**, and **Banzai Cloud Kafka Operator** simplify:

- Deployment and upgrades
- TLS and SCRAM authentication
- Monitoring with Prometheus/Grafana
- Kafka Topic and User CRDs

Example with Strimzi:

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
name: my-cluster
spec:
kafka:
version: 3.5.0
replicas: 3
storage:
type: persistent-claim
size: 100Gi
class: gp2
zookeeper:
replicas: 3
storage:
type: persistent-claim
size: 50Gi
```

---

#### 4. Persistent Storage Planning

Kafka requires low-latency, high-throughput disks. Best practices:

- Use **SSD-backed storage** (e.g., gp2, io1)
- Avoid ephemeral storage (e.g., `emptyDir`)
- Use **ReadWriteOnce** PVCs with StatefulSets
- Monitor disk usage and set **log retention policies** (e.g., `log.retention.hours`)

---

#### 5. Optimize Broker Configuration

Tune these settings for containerized environments:

```properties
broker.id=${HOSTNAME##*-}
log.dirs=/var/lib/kafka/data
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://${POD_NAME}.kafka:9092
num.network.threads=3
num.io.threads=8
num.partitions=3
log.retention.hours=72
log.segment.bytes=1073741824
```

Use Kubernetes environment variables or ConfigMaps to inject dynamic values.

---

#### 6. Secure Kafka Communications

Secure your Kafka setup using:

- **TLS encryption** for broker-to-broker and client communication
- **SASL (SCRAM or OAuth)** for authentication
- **RBAC-based topic access control**

Strimzi and Confluent Operator provide easy TLS and user secret management.

---

#### 7. Monitor and Alert

Monitor Kafka health and performance with:

- **Prometheus** exporters (JMX Exporter)
- **Grafana dashboards**
- **Liveness/Readiness probes** on broker ports

Track key metrics:

- Under-replicated partitions
- ISR shrinkage
- Broker disk usage
- Request latencies

---

#### 8. Enable Horizontal and Vertical Scaling

- Scale **brokers** by increasing StatefulSet replicas
- Tune **Kafka topic partition count** to leverage new brokers
- Resize **PVCs** or use **StorageClasses** with expansion support
- Auto-scale **producers/consumers** based on lag using HPA or KEDA

Note: Kafka brokers are **not stateless**, so **partition reassignment** may be needed after scaling.

---

#### Conclusion

Running Kafka on Kubernetes is no longer an experimental setup — it’s now a production-ready deployment model that offers **automation**, **portability**, and **elasticity**. With the right combination of StatefulSets, persistent storage, service discovery, and operators, you can run a **fault-tolerant, secure, and scalable Kafka cluster** natively in your Kubernetes environment.

Follow these best practices to simplify operations, optimize performance, and enable real-time event streaming at cloud scale.
