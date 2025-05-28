---
layout: post
title: Automating Pulsar Cluster Management with Kubernetes Operators
subtitle: Simplify deployment and scaling of Apache Pulsar with Kubernetes-native automation using Pulsar Operators
categories: Pulsar
tags: [Pulsar, Kubernetes, Operators, Automation, DevOps, Cloud Native, Pulsar Operator, Stream Processing]
excerpt: Learn how to automate Apache Pulsar cluster operations using Kubernetes Operators. Explore benefits, deployment strategies, and custom resource management for efficient, scalable Pulsar administration.
---
Managing distributed systems like **Apache Pulsar** can be complex — especially at scale. Tasks such as provisioning brokers, configuring BookKeeper, managing Zookeeper, and ensuring high availability often require deep operational expertise.

Enter the **Kubernetes Operator pattern**, which brings **Kubernetes-native automation** to Pulsar. With the **Pulsar Operator**, you can declaratively manage your Pulsar clusters, automate rollouts and upgrades, and scale components on demand.

In this post, we’ll explore how Kubernetes Operators simplify Pulsar cluster operations, how to deploy them, and best practices for production-grade deployments.

---

#### What is a Kubernetes Operator?

A **Kubernetes Operator** is a controller that extends Kubernetes APIs to manage the lifecycle of complex, stateful applications.

Key capabilities:
- Manages custom resources (CRDs)
- Monitors cluster state continuously
- Automates tasks like provisioning, scaling, healing, and upgrades

For Apache Pulsar, the operator handles:
- Brokers
- Bookies (Apache BookKeeper)
- Zookeeper clusters
- Proxies
- Configuration and secret management

---

#### Benefits of Using the Pulsar Operator

- **Declarative configuration** using YAML
- **Automated upgrades** of Pulsar components
- **Horizontal scaling** of brokers and bookies
- **Built-in observability** with Prometheus and Grafana integration
- **Secure handling** of secrets and TLS certificates
- Seamless integration with **Helm**, **CI/CD**, and GitOps workflows

---

#### Deploying the Pulsar Operator

##### Step 1: Install the Operator

You can install the Pulsar Operator using Helm:

```bash
helm repo add streamnative https://charts.streamnative.io
helm repo update

helm install pulsar-operator streamnative/pulsar-operator \
--namespace pulsar-system --create-namespace
```

Alternatively, you can use `kubectl apply` with the provided CRDs and manifests.

---

##### Step 2: Define a PulsarCluster Custom Resource

Create a `PulsarCluster` CRD that defines your cluster layout:

```yaml
apiVersion: pulsar.streamnative.io/v1alpha1
kind: PulsarCluster
metadata:
name: pulsar-prod
namespace: pulsar
spec:
zookeeper:
replicas: 3
bookkeeper:
replicas: 3
broker:
replicas: 3
proxy:
replicas: 2
autoRecovery:
enabled: true
monitoring:
prometheus: true
grafana: true
```

Apply the CR:

```bash
kubectl apply -f pulsar-cluster.yaml
```

The operator will automatically deploy and wire up Zookeeper, BookKeeper, Brokers, and Proxies.

---

#### Scaling the Cluster

Update the custom resource to scale:

```yaml
spec:
broker:
replicas: 6
```

Then re-apply:

```
kubectl apply -f pulsar-cluster.yaml
```

The operator handles rolling updates and stateful pod identity preservation.

---

#### TLS and Secret Management

Enable TLS for secure communication:

```yaml
spec:
tls:
enabled: true
secretName: pulsar-tls
```

Use Kubernetes secrets to mount certs securely.

---

#### Monitoring and Observability

The operator can deploy:
- **Prometheus** exporters for brokers, bookies, zookeeper
- **Grafana dashboards** out-of-the-box
- **Alerting rules** for common failure scenarios

This simplifies operational insights and troubleshooting.

---

#### Best Practices

- Use **persistent volumes (PVCs)** for BookKeeper and Zookeeper
- Deploy in **separate namespaces** for isolation
- Leverage **HPA** (Horizontal Pod Autoscaling) for load-driven scaling
- Integrate with **cert-manager** for auto-renewed TLS
- Monitor logs using **ELK stack** or **Fluent Bit**

---

#### Real-World Use Cases

- **Multi-tenant SaaS**: Spin up separate Pulsar clusters per customer with isolated namespaces
- **CI/CD Integration**: GitOps pipelines automatically reconcile Pulsar configuration via CRDs
- **Cloud-Native Migration**: Lift-and-shift on-prem Pulsar to Kubernetes with minimal disruption

---

#### Conclusion

Apache Pulsar is a powerful messaging platform, and with the help of **Kubernetes Operators**, it becomes **easy to deploy, scale, and manage** in cloud-native environments.

The **Pulsar Operator** eliminates manual toil and unlocks seamless automation, enabling your team to focus on building real-time data products — not babysitting infrastructure.

Whether you're operating in hybrid cloud, public cloud, or on-prem Kubernetes, this approach brings **reliability, consistency, and DevOps harmony** to Pulsar cluster management.
