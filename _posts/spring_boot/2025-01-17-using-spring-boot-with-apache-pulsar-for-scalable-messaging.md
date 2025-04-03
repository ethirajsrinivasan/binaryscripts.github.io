---
layout: post
title: Using Spring Boot with Apache Pulsar for Scalable Messaging
subtitle: Build scalable and resilient messaging systems using Spring Boot and Apache Pulsar
categories: Spring Boot
tags: [Java, Spring Boot, Apache Pulsar, Messaging, Event-Driven, Microservices]
excerpt: Learn how to integrate Apache Pulsar with Spring Boot to build scalable, event-driven messaging systems. This guide covers configuration, producers, consumers, topics, and performance tuning.
---



In today’s microservices and real-time data architectures, **messaging systems** play a critical role in decoupling services, improving responsiveness, and enabling event-driven design. While Kafka has been the go-to tool for many, **Apache Pulsar** is emerging as a powerful alternative, offering **multi-tenancy, geo-replication**, and **true queue + stream semantics**.

This post shows how to integrate **Apache Pulsar** with **Spring Boot**, covering producers, consumers, topics, and key best practices for building **scalable messaging systems**.

---

#### Why Choose Apache Pulsar?

Apache Pulsar is a distributed messaging and streaming platform with some key differentiators:

- **Separation of storage and compute**
- **Multi-tenancy** with authentication and authorization
- **Geo-replication** out of the box
- Support for both **streaming (pub-sub)** and **queue (exclusive/failover)** patterns
- Built-in schema registry

These features make Pulsar suitable for both simple event queues and large-scale data pipelines.

---

#### Setting Up the Environment

You can run a standalone Pulsar cluster using Docker:

```bash
docker run -it -p 6650:6650 -p 8080:8080 apachepulsar/pulsar:3.1.0 bin/pulsar standalone
```

- Broker Port: `6650`
- Admin HTTP Port: `8080`

Access the admin console: `http://localhost:8080`

---

#### Adding Dependencies to Spring Boot

Spring has native support for Pulsar via the **Spring for Apache Pulsar** project.

Add the following dependencies in `pom.xml`:

```xml
<dependency>
<groupId>org.springframework.pulsar</groupId>
<artifactId>spring-pulsar-spring-boot-starter</artifactId>
<version>0.2.0</version>
</dependency>
```

---

#### Configuration in application.yml

Configure the broker service URL and other client settings:

```yml
spring:
pulsar:
client:
service-url: pulsar://localhost:6650
producer:
topic-name: my-topic
consumer:
topic-names: my-topic
subscription-name: my-subscription
subscription-type: exclusive
```

---

#### Creating a Pulsar Producer

You can auto-wire a `PulsarTemplate` to send messages:

```java
@Service
public class NotificationProducer {

    private final PulsarTemplate<String> pulsarTemplate;

    public NotificationProducer(PulsarTemplate<String> pulsarTemplate) {
        this.pulsarTemplate = pulsarTemplate;
    }

    public void send(String message) {
        pulsarTemplate.send("notifications", message);
    }
}
```

Pulsar supports multiple serialization formats like JSON, Avro, and Protobuf.

---

#### Consuming Messages from a Topic

Use the `@PulsarListener` annotation to consume messages:

```java
@Component
public class NotificationConsumer {

    @PulsarListener(subscriptionName = "notification-sub", topics = "notifications")
    public void receive(String message) {
        System.out.println("Received: " + message);
    }
}
```

Subscription types:
- **Exclusive** (only one consumer)
- **Shared** (multiple consumers share load)
- **Failover** (primary/backup)
- **Key_Shared** (hashing per key)

---

#### Using POJOs and JSON Schema

Pulsar supports schema evolution. You can send Java objects instead of raw strings:

```java
public class Order {
private String id;
private double amount;
}
```

```java
pulsarTemplate.newMessage(Order.class)
.withTopic("orders")
.withValue(new Order("123", 500.0))
.send();
```

Configure the listener:

```java
@PulsarListener(topics = "orders", subscriptionName = "order-sub")
public void handleOrder(Order order) {
System.out.println("Order received: " + order.getId());
}
```

---

#### Scaling Consumers and Partitioned Topics

To scale consumers, use **partitioned topics**:

```bash
bin/pulsar-admin topics create-partitioned-topic persistent://public/default/my-topic -p 3
```

In Spring:

```yml
spring:
pulsar:
consumer:
subscription-type: shared
```

Each consumer instance receives a subset of messages, enabling parallelism.

---

#### Error Handling and Retries

Use `@PulsarListener` error handling options:

```java
@PulsarListener(topics = "errors", subscriptionName = "err-sub")
public void onError(String msg) {
try {
process(msg);
} catch (Exception ex) {
throw new PulsarListenerException("Failed to process", ex);
}
}
```

Configure retries and dead-letter topics in YAML:

```yml
spring:
pulsar:
consumer:
dead-letter-policy:
max-redeliver-count: 3
dead-letter-topic: errors-DLQ
```

---

#### Monitoring and Metrics

Monitor your Pulsar cluster with:
- **Pulsar Manager** UI
- **Prometheus + Grafana**
- **pulsar-admin** CLI tool

Expose metrics from Spring Boot using Micrometer and integrate them with Prometheus.

---

#### Conclusion

**Apache Pulsar + Spring Boot** provides a powerful platform for building **scalable**, **event-driven**, and **resilient** messaging systems. With native support for schemas, partitioned topics, and flexible subscription models, Pulsar addresses the limitations of traditional brokers.

Whether you're modernizing monoliths, building microservices, or designing IoT/event-processing pipelines, Pulsar’s architecture gives you the tools to build systems that scale.
