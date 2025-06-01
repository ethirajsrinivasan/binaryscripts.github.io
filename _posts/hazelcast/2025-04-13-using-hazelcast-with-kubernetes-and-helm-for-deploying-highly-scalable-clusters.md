---
layout: post
title: Deploying Highly Scalable Hazelcast Clusters with Kubernetes and Helm
subtitle: A technical guide to leveraging Hazelcast, Kubernetes, and Helm for robust, scalable in-memory data grids
categories: Hazelcast
tags: [Hazelcast, Kubernetes, Helm, Distributed Systems, In-Memory Data Grid, Scalability, Cloud Native, Big Data]
excerpt: Learn how to deploy highly scalable Hazelcast clusters using Kubernetes and Helm charts. This guide dives deep into advanced configurations, scaling strategies, and best practices for cloud-native in-memory data grids.
---
Hazelcast is a powerful in-memory data grid solution that excels in providing distributed caching, computing, and storage capabilities. When combined with Kubernetes, it offers a cloud-native approach to deploying scalable, resilient Hazelcast clusters. Kubernetes orchestrates containerized applications, ensuring *high availability* and *dynamic scaling*, which aligns perfectly with Hazelcast’s distributed nature.

In this post, we explore how to deploy Hazelcast clusters using Kubernetes and Helm, focusing on advanced use cases and configuration strategies that cater to intermediate and advanced users aiming to maximize cluster performance and scalability.

#### Why Use Helm for Hazelcast Deployment?

Helm simplifies Kubernetes application management by packaging Kubernetes resources into charts. Using Helm for Hazelcast deployments brings several benefits:

- **Version Control:** Easily manage and upgrade Hazelcast deployments.
- **Parameterization:** Customize Hazelcast configurations like cluster size, network settings, and persistence via Helm values.
- **Repeatability:** Deploy identical Hazelcast clusters across different environments with minimal effort.
- **Scalability:** Adjust cluster size dynamically by modifying Helm values and applying upgrades.

Helm charts for Hazelcast are maintained actively by the community and Hazelcast itself, offering production-ready templates optimized for various deployment scenarios.

#### Preparing Your Kubernetes Environment

Before deploying Hazelcast, ensure your Kubernetes cluster meets the following prerequisites:

- Kubernetes version 1.20 or higher (for compatibility with Helm 3 and latest APIs).
- Adequate resource quotas to handle Hazelcast pods, typically CPU and memory to support JVM-based workloads.
- Storage classes configured if using persistence or Hot Restart features.
- RBAC permissions for Helm and Hazelcast operators (if using the Hazelcast Operator).

Install Helm CLI and add the Hazelcast Helm repository:

```shell
helm repo add hazelcast https://hazelcast-charts.s3.amazonaws.com/
helm repo update
```

#### Deploying Hazelcast Cluster with Helm

To deploy a basic Hazelcast cluster, customize the `values.yaml` or pass parameters directly with Helm:

```shell
helm install my-hazelcast hazelcast/hazelcast \
  --set cluster.members=5 \
  --set hazelcast.persistence.enabled=true \
  --set hazelcast.network.port=5701
```

Key configuration parameters to consider:

- `cluster.members`: Number of Hazelcast nodes to run.
- `hazelcast.persistence.enabled`: Enables Hot Restart for data durability.
- `hazelcast.network.port`: Default Hazelcast port for member communication.
- `hazelcast.jmx.enabled`: Enable JMX metrics for monitoring.
- `hazelcast.resources.requests` and `hazelcast.resources.limits`: Define pod resource constraints.

#### Advanced Hazelcast Configuration on Kubernetes

##### Network Configuration and Discovery

By default, Hazelcast uses multicast for member discovery, which is often disabled in Kubernetes environments. Instead, Kubernetes DNS or Hazelcast’s Kubernetes plugin should be used.

In your Helm values, enable Kubernetes discovery:

```yaml
hazelcast:
  network:
    join:
      kubernetes:
        enabled: true
        service-name: my-hazelcast-hazelcast
```

This ensures Hazelcast members discover each other using Kubernetes services, enabling seamless scaling and failover.

##### StatefulSets and Persistent Volumes

For persistence and stable network identities, deploy Hazelcast using StatefulSets with Persistent Volume Claims (PVCs):

```yaml
hazelcast:
  persistence:
    enabled: true
  statefulSet:
    enabled: true
  volumeClaimTemplates:
    - metadata:
        name: hazelcast-data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

StatefulSets guarantee pod ordering and stable hostnames, crucial for Hot Restart and data recovery.

##### Autoscaling Hazelcast Clusters

While Kubernetes Horizontal Pod Autoscaler (HPA) can scale pods based on CPU or memory, Hazelcast cluster size also depends on data partitioning and member stability.

Use a combination of:

- **Kubernetes HPA** for pod-level scaling.
- **Hazelcast Management Center** to monitor cluster health and rebalance partitions.
- **Custom Metrics Adapter** to enable autoscaling based on Hazelcast-specific metrics like cluster size or partition load.

Example HPA config snippet:

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hazelcast-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: StatefulSet
    name: my-hazelcast
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

#### Monitoring and Logging Best Practices

Integrate Hazelcast with Kubernetes monitoring tools for observability:

- Enable **JMX metrics** and expose via Prometheus JMX exporter.
- Use **Grafana dashboards** tailored for Hazelcast cluster performance (e.g., partition distribution, heap memory, CPU usage).
- Collect logs via Kubernetes logging drivers or sidecars for centralized management.

Proper monitoring helps preemptively identify scaling needs and troubleshoot cluster issues.

#### Security Considerations

Secure your Hazelcast Kubernetes deployment by:

- Enabling **TLS encryption** for member-to-member communication.
- Using **Kubernetes Secrets** to store sensitive configurations like certificates and passwords.
- Implementing RBAC policies to restrict access to Hazelcast pods and services.
- Running Hazelcast pods with **non-root users** and least privilege security contexts.

These measures ensure your distributed data grid remains protected in multi-tenant environments.

#### Conclusion

Deploying Hazelcast clusters with Kubernetes and Helm empowers teams to build *highly scalable*, *resilient*, and *cloud-native* data grid solutions. Leveraging Helm charts simplifies configuration management and cluster lifecycle, while Kubernetes provides robust orchestration and scaling capabilities.

By fine-tuning network discovery, persistence, autoscaling, and security, advanced users can harness the full potential of Hazelcast for real-time data processing and caching in modern distributed systems.

Start experimenting with the Hazelcast Helm charts today to unlock scalable in-memory data grids that grow with your application demands.
