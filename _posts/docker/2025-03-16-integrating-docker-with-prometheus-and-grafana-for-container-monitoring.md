---
layout: post
title: Integrating Docker with Prometheus and Grafana for Container Monitoring
subtitle: Set up Prometheus and Grafana to monitor Docker containers with real-time metrics and dashboards
categories: Docker
tags: [Docker, Prometheus, Grafana, Monitoring, DevOps, Observability, Container Metrics]
excerpt: Learn how to monitor Docker containers using Prometheus and Grafana. Set up exporters, collect metrics, and create insightful dashboards to ensure container health and performance.
---
As applications shift toward microservices and containerized environments, monitoring becomes a cornerstone of **DevOps and SRE practices**. Tools like **Prometheus** and **Grafana** provide robust and customizable monitoring capabilities for Docker containers — enabling real-time observability, performance tracking, and alerting.

This guide walks you through **integrating Docker with Prometheus and Grafana**, including installation, configuration, metrics collection, and dashboard visualization.

---

#### Why Use Prometheus and Grafana for Docker Monitoring?

**Prometheus** is an open-source monitoring system that collects metrics via a pull model and stores time-series data. **Grafana** provides interactive dashboards and visualizations using Prometheus as a backend.

Benefits include:
- Lightweight and open-source
- Fine-grained container metrics (CPU, memory, disk I/O, network)
- Customizable alerts
- Real-time dashboards

---

#### Architecture Overview

```
[Docker Daemon] → [cAdvisor / node-exporter]
↓
[Prometheus Server]
↓
[Grafana]
```

- **cAdvisor**: Exposes Docker container stats
- **node-exporter**: Exposes host-level metrics
- **Prometheus**: Scrapes and stores metrics
- **Grafana**: Visualizes metrics and supports alerting

---

#### Step 1: Set Up Docker Monitoring Stack with Docker Compose

Create a `docker-compose.yml` file:

```yml
version: '3'

services:
prometheus:
image: prom/prometheus
volumes:
- ./prometheus.yml:/etc/prometheus/prometheus.yml
ports:
- "9090:9090"

cadvisor:
image: gcr.io/cadvisor/cadvisor:latest
ports:
- "8080:8080"
volumes:
- /:/rootfs:ro
- /var/run:/var/run:ro
- /sys:/sys:ro
- /var/lib/docker/:/var/lib/docker:ro

grafana:
image: grafana/grafana
ports:
- "3000:3000"
environment:
- GF_SECURITY_ADMIN_PASSWORD=admin
volumes:
- grafana-storage:/var/lib/grafana

volumes:
grafana-storage:
```

---

#### Step 2: Configure Prometheus to Scrape cAdvisor

Create `prometheus.yml`:

```yml
global:
scrape_interval: 15s

scrape_configs:
- job_name: 'cadvisor'
  static_configs:
  - targets: ['cadvisor:8080']
    ```

Place this file in the same directory as your Docker Compose file.

---

#### Step 3: Start the Monitoring Stack

Launch everything:

```bash
docker-compose up -d
```

Verify:
- Prometheus at `http://localhost:9090`
- Grafana at `http://localhost:3000` (admin/admin login)

---

#### Step 4: Import Grafana Docker Monitoring Dashboard

In Grafana:
- Add a **Prometheus data source** (`http://prometheus:9090`)
- Import community dashboard ID **893** or search “Docker” in the Grafana dashboard marketplace

This gives a complete overview of:
- CPU and memory usage per container
- Network I/O
- Block I/O
- Container restarts and uptime

---

#### Step 5: Add Alerts and Anomaly Detection

Create Grafana alerts:
- High CPU for > 5 mins
- Memory usage > 90%
- Container not running

Example rule:

```
WHEN avg() OF query(A, 5m, now) IS ABOVE 80
SEND TO Slack / Email / PagerDuty
```

---

#### Optional: Monitor Host Metrics with Node Exporter

Add to your `docker-compose.yml`:

```yml
node-exporter:
image: prom/node-exporter
ports:
- "9100:9100"
volumes:
- /proc:/host/proc:ro
- /sys:/host/sys:ro
- /:/rootfs:ro
```

Update Prometheus config:

```yml
- job_name: 'node'
  static_configs:
  - targets: ['node-exporter:9100']
    ```

This enables host-level observability alongside container metrics.

---

#### Best Practices

- Use **labels and tags** to organize metrics by service/environment
- Set **retention policies** in Prometheus to manage disk usage
- Secure endpoints using **basic auth** or **reverse proxies**
- Backup Grafana dashboards regularly
- Use **Grafana provisioning** to automate dashboard deployment

---

#### Conclusion

By integrating **Docker with Prometheus and Grafana**, you gain powerful, real-time insights into your containerized workloads. This stack provides full observability into performance, failures, and resource usage — critical for proactive infrastructure monitoring.

Whether you’re running a dev environment or a production-scale microservices platform, this setup forms the foundation for **robust, scalable container monitoring**.
