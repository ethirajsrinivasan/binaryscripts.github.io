---
layout: post
title: Using Java with Apache Pulsar for Stream Processing
subtitle: Build scalable real-time stream processing applications using Java and Apache Pulsar
categories: Java
tags: [Java, Apache Pulsar, Stream Processing, Messaging, Real-Time, Big Data]
excerpt: Learn how to use Apache Pulsar with Java to build high-throughput, low-latency stream processing systems. Understand producers, consumers, subscriptions, and Pulsar’s multi-topic architecture.
---
In a data-driven world, stream processing enables real-time decision-making, fraud detection, event sourcing, and analytics. **Apache Pulsar**, a next-generation distributed messaging and streaming platform, is gaining popularity for its **multi-tenancy**, **scalability**, and **low latency**.

This guide walks you through how to use **Java** to produce and consume messages with Apache Pulsar, build real-time processing pipelines, and leverage Pulsar’s unique features like **multi-topic subscriptions**, **message replay**, and **horizontal scalability**.

---

#### Why Apache Pulsar?

Apache Pulsar offers powerful features that go beyond traditional messaging systems like Kafka or RabbitMQ:

- **Built-in multi-tenancy and geo-replication**
- **Decoupled storage and serving layers**
- **Durable messaging with message acknowledgments**
- **Multiple subscription types: exclusive, shared, failover**
- **Native support for event time and message reprocessing**

Its architecture allows for scaling publishers and consumers independently — perfect for **Java-based microservices**.

---

#### Setting Up Pulsar with Java

Add the Pulsar client library to your Maven project:

```xml
<dependency>
<groupId>org.apache.pulsar</groupId>
<artifactId>pulsar-client</artifactId>
<version>3.1.0</version>
</dependency>
```

You can run Pulsar locally using Docker:

```bash
docker run -it -p 6650:6650 -p 8080:8080 apachepulsar/pulsar:3.1 bin/pulsar standalone
```

This runs a standalone Pulsar broker suitable for development and testing.

---

#### Creating a Pulsar Producer in Java

Producers send messages to topics. Here’s a simple example:

```java
PulsarClient client = PulsarClient.builder()
.serviceUrl("pulsar://localhost:6650")
.build();

Producer<String> producer = client.newProducer(Schema.STRING)
.topic("orders")
.create();

producer.send("Order received: ID#12345");
client.close();
```

You can configure batching, compression, and message routing via the producer builder.

---

#### Creating a Pulsar Consumer in Java

Consumers subscribe to topics and process messages asynchronously or synchronously.

```java
Consumer<String> consumer = client.newConsumer(Schema.STRING)
.topic("orders")
.subscriptionName("order-service")
.subscriptionType(SubscriptionType.Exclusive)
.subscribe();

Message<String> msg = consumer.receive();
System.out.println("Received: " + msg.getValue());
consumer.acknowledge(msg);
```

Change the `SubscriptionType` to `Shared` or `Failover` to scale consumers or build fault-tolerant services.

---

#### Subscription Types in Pulsar

- **Exclusive**: Only one consumer allowed
- **Shared**: Multiple consumers can receive messages in round-robin
- **Failover**: Active/passive consumer model
- **Key_Shared**: Partitioned messages by key for order guarantees

Choosing the right subscription type is critical for balancing **throughput**, **ordering**, and **fault tolerance**.

---

#### Using Topics and Partitions

You can publish to:
- A single topic
- A list of topics
- Pattern-matched topics (`persistent://public/default/order-.*`)

For high volume, use **partitioned topics**:

```bash
bin/pulsar-admin topics create-partitioned-topic orders -p 4
```

Your Java producer will automatically distribute messages across partitions based on keys or round-robin.

---

#### Pulsar Functions for Stream Processing

You can deploy lightweight Java functions directly into Pulsar:

```java
public class OrderEnricher implements Function<String, String> {
public String process(String input, Context context) {
return input + " - enriched";
}
}
```

Deploy with:

```bash
pulsar-admin functions create \
--jar target/functions.jar \
--classname com.example.OrderEnricher \
--tenant public \
--namespace default \
--name enrich-orders \
--inputs orders \
--output enriched-orders
```

This enables stream processing without external infrastructure like Flink or Spark.

---

#### Use Cases in Java Applications

Java and Pulsar are well-suited for building:
- **Real-time fraud detection systems**
- **Event-driven microservices**
- **Analytics and monitoring dashboards**
- **Order/event log processors**
- **IoT ingestion pipelines**

---

#### Best Practices

- Use **message keys** for order-sensitive streams
- Enable **batching** for small messages to improve throughput
- Always **acknowledge messages** to avoid redelivery
- Handle **dead letter topics (DLTs)** for failure recovery
- Monitor performance with **Pulsar metrics and Grafana**

---

#### Conclusion

Apache Pulsar + Java is a powerful combination for building **resilient, scalable, real-time** systems. With built-in support for high throughput, message durability, and flexible subscriptions, Pulsar provides an edge over traditional messaging systems.

Whether you're scaling microservices or processing real-time data streams, Java and Pulsar can take your architecture to the next level.
