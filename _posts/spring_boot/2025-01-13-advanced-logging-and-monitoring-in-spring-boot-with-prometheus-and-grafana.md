---
layout: post
title: Advanced Logging and Monitoring in Spring Boot with Prometheus and Grafana
subtitle: Implement powerful observability in Spring Boot using Prometheus for metrics and Grafana for real-time visualization
categories: SpringBoot
tags: [Spring Boot, Prometheus, Grafana, Logging, Monitoring, Observability, Micrometer]
excerpt: Learn how to implement advanced logging and monitoring in Spring Boot applications using Prometheus and Grafana. Set up metrics, visualize performance, and build real-time dashboards for operational insights.
---
In production systems, having visibility into application behavior is critical. Without effective **monitoring** and **logging**, diagnosing failures or performance issues becomes guesswork.

**Spring Boot**, in combination with **Prometheus** and **Grafana**, provides a robust, open-source toolchain for **observability**. In this article, we’ll walk through setting up **metrics collection**, **custom monitoring**, and **visual dashboards** to track the health and performance of your Spring Boot application in real time.

---

#### Observability Stack Overview

To monitor modern applications, we typically collect:

- **Metrics**: Quantitative measurements like request count, latency, and memory usage
- **Logs**: Time-stamped events with contextual information
- **Traces**: End-to-end request paths across services

For this guide, we’ll focus on **metrics** with Prometheus and **visualization** using Grafana. Logging will also be enhanced using structured and contextual logs.

---

#### Micrometer: Metrics Facade for Spring Boot

Spring Boot integrates with **Micrometer**, which provides a vendor-neutral metrics API. It supports backends like Prometheus, Datadog, New Relic, and more.

Add the following to your `pom.xml`:

```xml
<dependency>
<groupId>io.micrometer</groupId>
<artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

Enable the Prometheus endpoint in `application.yml`:

```yml
management:
endpoints:
web:
exposure:
include: health,info,prometheus
endpoint:
prometheus:
enabled: true
```

Access metrics at `/actuator/prometheus`.

---

#### Defining Custom Metrics

Micrometer provides built-in metrics, but you can also define your own counters, gauges, and timers.

```java
@Autowired
private MeterRegistry meterRegistry;

private Counter customCounter;

@PostConstruct
public void init() {
customCounter = meterRegistry.counter("custom.api.calls");
}

public void someMethod() {
customCounter.increment();
}
```

You can also time operations:

```java
Timer.Sample sample = Timer.start(meterRegistry);
// do work
sample.stop(meterRegistry.timer("task.execution.time"));
```

---

#### Setting Up Prometheus

Download and run Prometheus, then add a `prometheus.yml` config:

```yml
scrape_configs:
- job_name: 'spring-boot-app'
  metrics_path: '/actuator/prometheus'
  static_configs:
  - targets: ['localhost:8080']
    ```

Start Prometheus:

```bash
./prometheus --config.file=prometheus.yml
```

Once running, go to `http://localhost:9090` and test metrics queries like:

```
http_server_requests_seconds_count
process_cpu_usage
custom_api_calls_total
```

---

#### Visualizing Metrics in Grafana

1. Install Grafana and start the server.
2. Log in at `http://localhost:3000` (default admin/admin).
3. Add Prometheus as a **data source**.
4. Create a new dashboard and panel.

Sample PromQL query for total HTTP requests:

```
sum(rate(http_server_requests_seconds_count[1m])) by (uri)
```

Visualize custom metrics, JVM health (heap, GC), and API latencies.

---

#### Advanced Logging with Logback

Structured logging improves log usability in centralized platforms (ELK, Loki, etc.)

Example logback config:

```xml
<encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
```

Add contextual fields in your logs:

```java
MDC.put("userId", userId);
log.info("User request received");
MDC.clear();
```

Use tools like:
- **Fluentd** or **Filebeat** for shipping logs
- **ELK stack** or **Loki** for storage and querying

---

#### Adding Health Checks and Alerts

Spring Boot provides health indicators via Actuator:

```yml
management:
endpoint:
health:
show-details: always
```

Expose custom health indicators:

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
@Override
public Health health() {
// check database connection
return Health.up().build();
}
}
```

Prometheus + Grafana can alert on health status:

```yml
groups:
- name: spring-alerts
  rules:
  - alert: HighLatency
    expr: http_server_requests_seconds_max > 2
    for: 1m
    labels:
    severity: critical
    annotations:
    summary: "High response time detected"
    ```

---

#### Best Practices

- Use **tagged metrics** for rich dimensional analysis
- Avoid high-cardinality labels (e.g., UUIDs in paths)
- Group metrics by functional domain (e.g., `order.service.requests`)
- Structure logs in **JSON** for better parsing and aggregation
- Limit log volume in high-throughput environments with sampling

---

#### Conclusion

**Spring Boot + Prometheus + Grafana** is a powerful stack for real-time monitoring and observability. With Micrometer's seamless integration, custom metric tracking, structured logs, and rich dashboards, developers can
