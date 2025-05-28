---
layout: post
title: Pulsar for Building Event Driven Microservices with Spring Boot
subtitle: Integrate Apache Pulsar with Spring Boot to build reactive and scalable event-driven microservices
categories: Pulsar
tags: [Apache Pulsar, Spring Boot, Microservices, Event-Driven Architecture, Messaging, Reactive Systems]
excerpt: Learn how to use Apache Pulsar with Spring Boot to create scalable event-driven microservices. Explore architecture patterns, integration techniques, and real-world examples using Pulsar client libraries and Spring frameworks.
---
Event-driven architecture (EDA) has become a cornerstone of modern microservice design, enabling systems to react to changes asynchronously and scale independently. **Apache Pulsar**, with its high-throughput, low-latency, and multi-tenant capabilities, is an ideal messaging backbone for event-driven microservices. Combined with **Spring Boot**, developers can quickly build, deploy, and scale reactive services that communicate using events.

This post walks through how to use **Apache Pulsar with Spring Boot** to build reliable, loosely coupled microservices, and outlines best practices for integration, deployment, and observability.

---

#### Why Use Pulsar for Microservices?

- **Asynchronous communication** between services
- **Multi-tenancy** and topic-level isolation
- Built-in **message persistence and replay**
- Native support for **pub-sub and message queue semantics**
- **Backpressure and flow control** out of the box

Compared to traditional REST APIs, Pulsar enables **non-blocking**, **scalable**, and **resilient** inter-service communication.

---

#### Architecture Overview

```
[Service A] → (produces) → [Pulsar Topic: orders] → (consumes) → [Service B]
↘                        ↘
[Service C]             [Analytics Service]
```

Services publish and subscribe to Pulsar topics, allowing **event fan-out**, **decoupling**, and **reactive workflows**.

---

#### Integrating Pulsar with Spring Boot

You can integrate Pulsar using:

1. **Spring for Apache Pulsar** (`spring-pulsar`)
2. **Pulsar Java Client** directly

##### Maven Dependency

```xml
<dependency>
<groupId>org.apache.pulsar</groupId>
<artifactId>pulsar-client</artifactId>
<version>3.2.0</version>
</dependency>
<dependency>
<groupId>org.springframework.pulsar</groupId>
<artifactId>spring-pulsar</artifactId>
<version>0.2.0</version>
</dependency>
```

---

#### Example: Publishing Events from a Spring Boot Service

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    private final PulsarTemplate<String> pulsarTemplate;

    public OrderController(PulsarTemplate<String> pulsarTemplate) {
        this.pulsarTemplate = pulsarTemplate;
    }

    @PostMapping
    public ResponseEntity<String> placeOrder(@RequestBody Order order) {
        pulsarTemplate.send("orders", order.toString());
        return ResponseEntity.ok("Order placed");
    }
}
```

---

#### Example: Consuming Events

```java
@Component
public class OrderListener {

    @PulsarListener(topics = "orders", subscriptionName = "order-service-sub")
    public void handleOrder(String message) {
        System.out.println("Received order: " + message);
        // Process order
    }
}
```

This allows services to **react to published events** in real time.

---

#### Managing Schema and Data Formats

Pulsar supports:
- JSON
- Avro
- Protobuf
- Native schema registry

For Spring Boot, use Jackson for serialization:

```java
@Bean
public PulsarTemplate<Order> orderTemplate(PulsarClient client) {
Schema<Order> schema = Schema.JSON(Order.class);
ProducerBuilder<Order> builder = client.newProducer(schema).topic("orders");
return new PulsarTemplate<>(builder);
}
```

---

#### Error Handling and Retries

- Use **DLQs (Dead Letter Topics)** for failed messages
- Enable **retry policies** in `@PulsarListener`
- Add logging and metrics using Spring’s `@EventListener` hooks

```java
@PulsarListener(topics = "payments", subscriptionName = "retry-sub", deadLetterPolicy = @DeadLetterPolicy(
maxRedeliverCount = 3, deadLetterTopic = "payments-dlq"))
public void processPayment(String payload) {
// Handle payment or throw exception
}
```

---

#### Observability and Monitoring

- Expose metrics via **Micrometer + Prometheus**
- Monitor:
  - Consumer lag
  - Topic throughput
  - Error rate and retries
- Use **Pulsar Manager**, **Prometheus**, or **Grafana** dashboards

---

#### Best Practices

✅ Use **dedicated topics per domain/event type**  
✅ Design **idempotent consumers** to handle replays  
✅ Apply **backpressure handling** via Pulsar consumer config  
✅ Use **schema evolution** for backward compatibility  
✅ Keep messages **small and serializable** (avoid sending large blobs)  
✅ Enable **tracing** with OpenTelemetry or Spring Sleuth

---

#### Conclusion

Apache Pulsar and Spring Boot together offer a modern, scalable foundation for building **event-driven microservices**. By decoupling services through Pulsar topics and handling data reactively, you gain the agility, resilience, and performance needed for today’s dynamic cloud-native applications.

Whether you're modernizing a monolith or launching a new reactive platform, Pulsar with Spring Boot gives you the tools to build fast, robust, and future-proof microservice architectures.
