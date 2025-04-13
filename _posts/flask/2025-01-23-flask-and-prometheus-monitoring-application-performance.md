---
layout: post
title: "Flask and Prometheus Monitoring: Enhancing Application Performance"
subtitle: "Learn how to monitor Flask applications using Prometheus for real-time performance insights"
categories: Flask
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: ["Flask", "Prometheus", "Monitoring", "Performance", "Metrics", "DevOps"]
excerpt: "Discover how to integrate Prometheus with Flask to monitor application performance, track metrics, and optimize API response times."
---
Monitoring is essential for ensuring **optimal performance and reliability** in web applications. **Prometheus**, an open-source monitoring system, provides **powerful metrics collection, querying, and alerting** capabilities.

In this guide, we will explore how to **integrate Prometheus with Flask** to monitor application performance, collect useful metrics, and visualize them with **Grafana**.

## Why Use Prometheus for Monitoring?

Prometheus is widely used due to its:

- **Pull-based metric collection** for efficient data retrieval.
- **Multi-dimensional data model** with labels for flexible querying.
- **Built-in alerting system** via Alertmanager.
- **Integration with Grafana** for rich visual dashboards.

## Setting Up Flask with Prometheus

### Installing Dependencies

First, install the required packages:

```sh
pip install flask prometheus_client
```

- `prometheus_client`: Provides Prometheus-compatible metrics for Flask.

### Creating a Basic Flask App with Prometheus

```python
from flask import Flask
from prometheus_client import start_http_server, Summary, Counter, Gauge, Histogram
import time
import random

app = Flask(__name__)

# Define Prometheus metrics
REQUEST_COUNT = Counter("flask_request_count", "Total request count")
REQUEST_LATENCY = Histogram("flask_request_latency_seconds", "Request processing time")
ACTIVE_USERS = Gauge("flask_active_users", "Current active users")

@app.route("/")
def home():
REQUEST_COUNT.inc()  # Increment request count
start_time = time.time()

    # Simulate processing time
    time.sleep(random.uniform(0.1, 0.5))  
    
    REQUEST_LATENCY.observe(time.time() - start_time)  # Record latency
    return "Hello, Flask with Prometheus!"

if __name__ == "__main__":
start_http_server(8000)  # Start Prometheus metrics server
app.run(debug=True)
```

- The **Counter** tracks total requests.
- The **Histogram** records request latency.
- The **Gauge** measures active users.
- The **start_http_server(8000)** exposes metrics at `http://localhost:8000`.

## Exposing Metrics Endpoint

Prometheus needs an endpoint to scrape metrics. Update the Flask app:

```python
from prometheus_client import generate_latest, CONTENT_TYPE_LATEST

@app.route("/metrics")
def metrics():
return generate_latest(), 200, {"Content-Type": CONTENT_TYPE_LATEST}
```

- Now, Prometheus can scrape metrics from `http://localhost:5000/metrics`.

## Setting Up Prometheus

### Installing Prometheus

Download and install Prometheus:

```sh
wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-linux-amd64.tar.gz
tar -xvf prometheus-linux-amd64.tar.gz
cd prometheus-linux-amd64
```

### Configuring Prometheus

Modify `prometheus.yml`:

```yml
scrape_configs:
- job_name: "flask_app"
  metrics_path: "/metrics"
  static_configs:
  - targets: ["localhost:5000"]
    ```

Start Prometheus:

```sh
./prometheus --config.file=prometheus.yml
```

Access the **Prometheus UI** at `http://localhost:9090`.

## Visualizing Metrics with Grafana

### Installing Grafana

```sh
sudo apt-get install -y grafana
sudo systemctl start grafana-server
```

### Adding Prometheus as a Data Source

1. Open `http://localhost:3000` (default Grafana port).
2. Go to **Configuration > Data Sources**.
3. Select **Prometheus** and set the URL as `http://localhost:9090`.

### Creating a Dashboard

1. Click **Create > Dashboard**.
2. Add a **new panel** and use **PromQL queries** like:

  - `flask_request_count` (Total Requests)
  - `flask_request_latency_seconds` (Request Latency Histogram)
  - `flask_active_users` (Active Users)

## Setting Up Alerts

### Defining Alert Rules

Modify `prometheus.yml`:

```yml
alerting:
alertmanagers:
- static_configs:
- targets: ["localhost:9093"]

rule_files:
- "alert.rules.yml"
  ```

Create `alert.rules.yml`:

```yml
groups:
- name: flask_alerts
  rules:
  - alert: HighRequestLatency
    expr: flask_request_latency_seconds{quantile="0.9"} > 0.4
    for: 1m
    labels:
    severity: warning
    annotations:
    summary: "High request latency detected"
    ```

Start **Alertmanager**:

```sh
./alertmanager --config.file=alertmanager.yml
```

## Optimizing Flask for Performance

- **Use Gunicorn** for better concurrency:

  ```sh
  gunicorn -w 4 -b 0.0.0.0:5000 app:app
  ```

- **Enable Caching** with Redis or Memcached.
- **Use Async Frameworks** like **FastAPI** for high-performance APIs.

## Conclusion

Integrating **Prometheus with Flask** allows **real-time performance monitoring**, ensuring applications remain **fast and reliable**. By visualizing metrics in **Grafana** and setting up **alerts**, developers can **proactively address issues** and maintain system health.

🚀 **Start monitoring your Flask applications today with Prometheus and Grafana!**  
