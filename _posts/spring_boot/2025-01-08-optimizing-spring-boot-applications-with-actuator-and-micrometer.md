---
layout: post
title: Optimizing Spring Boot Applications with Actuator and Micrometer
subtitle: Use Actuator and Micrometer to monitor, tune, and optimize Spring Boot performance
categories: Spring Boot
tags: [Spring Boot, Micrometer, Actuator, Monitoring, Performance, Observability]
excerpt: Discover how to optimize Spring Boot applications using Actuator and Micrometer. Track metrics, monitor health, and improve performance with real-time observability and diagnostics.
---



Performance is a critical factor in modern applications. Whether you're serving millions of requests or running lightweight microservices, understanding how your **Spring Boot application** behaves in real-time is essential for stability and scalability.

Spring Boot provides two powerful tools to address this: **Spring Boot Actuator** and **Micrometer**. Combined, they offer deep insights into your application’s internals and expose metrics in a way that’s ready for modern monitoring tools like Prometheus and Grafana.

In this post, you’ll learn how to use Actuator and Micrometer to **monitor, measure, and optimize** your Spring Boot apps for production environments.

---

#### What is Spring Boot Actuator?

Spring Boot Actuator adds production-ready features to your application, including:

- Health checks
- Application metrics
- Thread dumps and heap dumps
- Environment and configuration introspection
- Custom endpoints

It exposes this information via **HTTP**, **JMX**, or **gRPC**, making it easy to integrate with external systems.

---

#### Getting Started with Spring Boot Actuator

Add the dependency:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Enable specific endpoints in `application.yml`:

```yml
management:
endpoints:
web:
exposure:
include: health,info,metrics,env,loggers
```

Once enabled, you can access endpoints like:

- `/actuator/health`
- `/actuator/metrics`
- `/actuator/env`

These give you deep introspection into the running application.

---

#### Introducing Micrometer for Metrics Collection

Micrometer is the **metrics facade** behind Spring Boot. It supports multiple monitoring backends like:

- Prometheus
- Graphite
- Datadog
- New Relic
- CloudWatch

Add Prometheus support via:

```xml
<dependency>
<groupId>io.micrometer</groupId>
<artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

Expose the Prometheus metrics endpoint:

```yml
management:
endpoints:
web:
exposure:
include: prometheus
```

Access metrics at: `http://localhost:8080/actuator/prometheus`

---

#### Collecting Custom Metrics

You can define your own application-level metrics:

```java
@Autowired
private MeterRegistry meterRegistry;

@PostConstruct
public void initMetrics() {
Gauge.builder("active.sessions", sessionService, SessionService::getActiveSessions)
.description("Number of active user sessions")
.register(meterRegistry);
}
```

You can also use counters:

```java
Counter counter = meterRegistry.counter("api.calls", "endpoint", "/orders");
counter.increment();
```

These metrics are automatically exported to the selected backend.

---

#### Monitoring Built-in Metrics

Micrometer captures numerous JVM and Spring metrics out of the box:

- JVM memory and GC (`jvm.memory.used`, `jvm.gc.pause`)
- CPU usage (`system.cpu.usage`)
- HTTP request performance (`http.server.requests`)
- Thread usage (`jvm.threads.live`, `jvm.threads.daemon`)
- Database and connection pool metrics (for HikariCP, etc.)

Use them to diagnose slow responses, memory leaks, or thread starvation.

---

#### Customizing Endpoint Security

Restrict access to Actuator endpoints:

```yml
management:
endpoints:
web:
exposure:
include: health, metrics
endpoint:
health:
show-details: when-authorized

spring:
security:
user:
name: admin
password: secret
```

Use Spring Security to require authentication for sensitive endpoints like `/env`, `/loggers`, or `/heapdump`.

---

#### Visualizing Metrics with Grafana and Prometheus

To get visual dashboards:

1. Run Prometheus to scrape metrics from `/actuator/prometheus`
2. Connect Grafana to Prometheus
3. Import a Spring Boot dashboard or create one using queries like:

```
rate(http_server_requests_seconds_count{uri="/api/products"}[1m])
```

This shows how many requests hit the `/api/products` endpoint per minute.

---

#### Performance Optimization Tips

- **Track long request durations**: Use `http.server.requests` histogram metrics
- **Tune database pool**: Monitor Hikari metrics to avoid connection bottlenecks
- **Monitor memory trends**: Use `jvm.memory.used` over time
- **Use alerts**: Set Prometheus alert rules for high GC time or error rates
- **Disable unused endpoints**: Expose only the necessary Actuator endpoints in production

---

#### Conclusion

Spring Boot Actuator and Micrometer empower you to turn your applications into **observable, debuggable, and tunable systems**. By exposing real-time metrics and health checks, you gain the visibility required to scale confidently and respond to production issues before users notice.

Whether you're deploying on bare metal, cloud platforms, or Kubernetes, integrating these tools gives you the control and insight to optimize performance and reliability.
