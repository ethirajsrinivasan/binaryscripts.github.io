---
layout: post
title: Kubernetes Custom Metrics and Horizontal Pod Autoscaling with Prometheus Adapter
subtitle: Deep dive into implementing custom metrics and autoscaling Kubernetes workloads using Prometheus Adapter
categories: Kubernetes
tags: [Kubernetes, Prometheus, Autoscaling, Custom Metrics, HPA, Monitoring, Cloud Native, DevOps]
excerpt: Learn how to leverage Kubernetes custom metrics and Horizontal Pod Autoscaling with Prometheus Adapter to build scalable, efficient applications on Kubernetes.
---
Horizontal Pod Autoscaling (HPA) is a vital Kubernetes feature that enables automatic scaling of pods based on observed metrics. While Kubernetes supports CPU and memory-based scaling out of the box, many real-world applications require **custom metrics** to drive scaling decisions — such as request latency, queue length, or business-specific KPIs.

This post explores how to implement **Kubernetes Custom Metrics API** integration using **Prometheus Adapter** to enable HPA based on custom metrics, empowering developers and operators to scale workloads dynamically and precisely.

---
The Kubernetes HPA controller relies on the **Metrics API** to retrieve metrics about pods or objects and adjusts replica counts accordingly. There are two main metrics APIs relevant for HPA:

- **Resource Metrics API:** Exposes standard metrics like CPU and memory usage.
- **Custom Metrics API:** Allows users to scale based on arbitrary application or infrastructure metrics exposed via Kubernetes APIs.

Prometheus, a popular open-source monitoring system, provides a rich source of time-series metrics. However, Kubernetes cannot consume Prometheus metrics natively for HPA. This gap is filled by the **Prometheus Adapter**.

---

#### What is Prometheus Adapter?

Prometheus Adapter acts as a bridge that exposes Prometheus metrics via the Kubernetes Custom Metrics API. It:

- Queries Prometheus for user-defined metrics.
- Transforms these metrics into a format compatible with Kubernetes Custom Metrics API.
- Enables HPA to fetch and use these metrics to scale pods dynamically.

Prometheus Adapter supports metrics from multiple sources, allowing flexible scaling strategies aligned with application needs.

---

#### Architecture Overview

1. **Prometheus Server** scrapes application and infrastructure metrics.
2. **Prometheus Adapter** queries Prometheus API based on configured rules.
3. **Kubernetes Metrics API** is extended by Prometheus Adapter to expose custom metrics.
4. **HPA Controller** requests metrics from the Metrics API and adjusts replicas accordingly.

![Prometheus Adapter Architecture](https://example.com/images/prometheus-adapter-architecture.png)

---

#### Setting Up Prometheus Adapter for Custom Metrics HPA

##### Step 1: Deploy Prometheus and Collect Metrics

Deploy Prometheus in your cluster (using Helm or manifests), and ensure your application exposes metrics in Prometheus format (commonly via `/metrics` endpoint).

##### Step 2: Install Prometheus Adapter

You can install Prometheus Adapter via Helm:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus-adapter prometheus-community/prometheus-adapter -n monitoring
```

##### Step 3: Configure Custom Metrics Mapping

Edit the `values.yaml` or adapter ConfigMap to map Prometheus metrics to Kubernetes custom metrics API. Example snippet:

```yaml
rules:
  custom:
    - seriesQuery: 'http_requests_total{namespace!="",pod!=""}'
      resources:
        overrides:
          namespace:
            resource: namespace
          pod:
            resource: pod
      name:
        matches: "^(.*)_total"
        as: "${1}_per_second"
      metricsQuery: 'sum(rate(http_requests_total{<<.LabelMatchers>>}[2m])) by (<<.GroupBy>>)'
```

This maps the `http_requests_total` metric from Prometheus to a custom metric that HPA can query.

---

#### Creating an HPA Based on Custom Metrics

Once the adapter exposes custom metrics, create an HPA object referencing those metrics. Example YAML:

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: custom-metric-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: 100
```

This HPA scales the `my-app` deployment based on average HTTP requests per second per pod.

---

#### Best Practices and Considerations

- **Metric Cardinality:** Keep metric labels minimal to avoid performance issues.
- **Metric Freshness:** Ensure Prometheus scraping intervals and adapter queries are tuned to reduce latency.
- **Security:** Use RBAC and TLS to secure Prometheus Adapter and Metrics API access.
- **Monitoring:** Track HPA events and pod scaling behavior to detect anomalies early.
- **Testing:** Simulate loads to validate scaling thresholds and response times.

---

#### Troubleshooting Tips

- Confirm Prometheus metrics are available and queryable via Prometheus UI.
- Check Prometheus Adapter logs for errors or misconfigurations.
- Verify Kubernetes API server has access to the adapter’s Custom Metrics API endpoints.
- Use `kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1"` to list exposed metrics.
- Ensure your application metrics expose the required labels (`namespace`, `pod`, etc.).

---

#### Conclusion

Integrating Kubernetes Custom Metrics API with Prometheus Adapter unlocks powerful autoscaling capabilities beyond CPU and memory. This setup enables **fine-grained, application-specific scaling**, essential for modern cloud-native workloads that require responsiveness to business and operational metrics.

By mastering Prometheus Adapter and custom metrics-driven HPA, Kubernetes users can optimize resource utilization, improve application resilience, and better manage cost and performance at scale.

