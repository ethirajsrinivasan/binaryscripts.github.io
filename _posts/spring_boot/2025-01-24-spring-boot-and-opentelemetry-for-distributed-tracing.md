---
layout: post
title: Spring Boot and OpenTelemetry for Distributed Tracing
subtitle: Monitor and trace distributed Java applications using Spring Boot and OpenTelemetry
categories: SpringBoot
tags: [Spring Boot, OpenTelemetry, Distributed Tracing, Observability, Java, Microservices]
excerpt: Learn how to instrument Spring Boot applications with OpenTelemetry for complete distributed tracing. Capture spans, correlate requests across services, and improve observability with ease.
---
Modern systems are composed of **distributed microservices**, often running across different environments, containers, or even cloud providers. When something goes wrong — latency spikes, failed requests, or inconsistent data — pinpointing the root cause can be extremely difficult.

This is where **distributed tracing** shines. With **OpenTelemetry**, the emerging industry standard for observability, you can track the entire lifecycle of a request as it flows through multiple services.

In this guide, we’ll walk through how to integrate **OpenTelemetry with Spring Boot**, enabling rich telemetry data, trace propagation, and compatibility with tools like Jaeger, Zipkin, or Grafana Tempo.

---

#### What is OpenTelemetry?

**OpenTelemetry (OTel)** is an open-source observability framework that provides:
- **Tracing**: Understand how requests propagate
- **Metrics**: Collect performance indicators
- **Logging**: Contextual logs with trace correlation

It standardizes instrumentation for multiple languages and allows you to send telemetry data to various backends using exporters.

---

#### Key Concepts

- **Trace**: A single end-to-end request across services
- **Span**: A unit of work within a trace (e.g., method call, DB query)
- **Context Propagation**: Carrying trace info across network calls
- **Exporter**: Sends telemetry data to observability platforms

---

#### Adding OpenTelemetry to Spring Boot

You can instrument Spring Boot apps manually or use the auto-instrumentation agent. The easiest way to get started is with the **OpenTelemetry Java Agent**.

##### Step 1: Download the Agent

Download the latest OpenTelemetry Java agent jar from:
[https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases](https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases)

##### Step 2: Run with the Agent

Add the agent to your startup script:

```bash
java -javaagent:opentelemetry-javaagent.jar \
-Dotel.service.name=springboot-app \
-Dotel.exporter.otlp.endpoint=http://localhost:4317 \
-jar target/myapp.jar
```

This automatically instruments HTTP clients, Spring MVC, JDBC, and more.

---

#### Manual Instrumentation with OpenTelemetry SDK

To manually create spans:

```xml
<dependency>
<groupId>io.opentelemetry</groupId>
<artifactId>opentelemetry-api</artifactId>
<version>1.28.0</version>
</dependency>
```

```java
Tracer tracer = GlobalOpenTelemetry.getTracer("com.example.MyService");

Span span = tracer.spanBuilder("processData").startSpan();
try (Scope scope = span.makeCurrent()) {
// business logic
span.setAttribute("custom.attribute", "value");
} finally {
span.end();
}
```

This helps when tracing custom logic outside the supported frameworks.

---

#### Exporting Traces to Jaeger or Zipkin

You can configure exporters programmatically or via environment variables.

**For Jaeger**:

```bash
-Dotel.exporter.jaeger.endpoint=http://localhost:14250
-Dotel.traces.exporter=jaeger
```

**For Zipkin**:

```bash
-Dotel.exporter.zipkin.endpoint=http://localhost:9411/api/v2/spans
-Dotel.traces.exporter=zipkin
```

Jaeger and Zipkin both provide UI interfaces for visualizing traces.

---

#### Propagating Traces Across Services

To correlate traces across services:
- Ensure each service uses the same propagation format (W3C by default)
- Use libraries like **RestTemplate**, **WebClient**, or **Feign** with the OTel agent to automatically propagate context

Manually propagate headers if needed:

```java
TextMapSetter<HttpHeaders> setter = HttpHeaders::set;
OpenTelemetry.getPropagators().getTextMapPropagator()
.inject(Context.current(), headers, setter);
```

---

#### Viewing Traces in Jaeger

After instrumentation:
1. Open [http://localhost:16686](http://localhost:16686)
2. Select your service name
3. View individual traces, spans, duration, and errors

This helps identify latency bottlenecks or failed downstream calls instantly.

---

#### Spring Boot with Micrometer and OpenTelemetry

Spring Boot 3+ supports **Micrometer Tracing**, which integrates OpenTelemetry under the hood:

```xml
<dependency>
<groupId>io.micrometer</groupId>
<artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
```

It works seamlessly with `@RestController`, `RestTemplate`, and `WebClient`, and supports both metrics and traces via the same config.

---

#### Security and Privacy Considerations

- Avoid recording sensitive data in spans
- Use **span filtering** to remove PII
- Limit span cardinality to reduce storage cost
- Rotate trace IDs periodically for security audits

---

#### Best Practices

- Start with auto-instrumentation for fast adoption
- Export to a local backend like Jaeger before integrating cloud tools
- Use `span attributes` and `events` for richer insights
- Correlate logs with traces using trace ID in log context
- Trace business-critical paths, not every method

---

#### Conclusion

Distributed tracing with **Spring Boot and OpenTelemetry** gives developers and DevOps teams visibility into how their applications behave under real-world conditions. With minimal setup, you gain powerful tools to trace requests across services, optimize performance, and troubleshoot issues faster.

As microservices architectures continue to grow, tracing is no longer optional — it's a must-have for **reliable, observable systems**.
