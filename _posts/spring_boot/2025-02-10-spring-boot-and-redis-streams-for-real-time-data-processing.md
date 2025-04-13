---
layout: post
title: Spring Boot and Redis Streams for Real-Time Data Processing
subtitle: Build scalable real-time data processing pipelines with Spring Boot and Redis Streams
categories: SpringBoot
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: [Spring Boot, Redis, Redis Streams, Real-Time, Messaging, Data Processing]
excerpt: Learn how to use Redis Streams with Spring Boot to build real-time data processing systems. Explore producers, consumers, consumer groups, and reliable message delivery patterns.
---
Real-time data processing is a critical component of modern applications. Whether you're handling financial transactions, IoT sensor readings, or event-based logs, you need a messaging system that is fast, reliable, and scalable.

**Redis Streams** is a powerful data structure introduced in Redis 5.0 that supports append-only log semantics with built-in message queues and consumer groups. Combined with **Spring Boot**, you can build **reactive, real-time pipelines** for ingesting, processing, and distributing event data efficiently.

This post explores how to integrate Redis Streams into Spring Boot applications for scalable real-time data processing.

---

#### What is Redis Streams?

Redis Streams is an **append-only log** that lets you:

- Publish messages to a stream
- Track consumers via **consumer groups**
- Acknowledge processed messages
- Replay or reprocess unacknowledged events
- Store high-throughput event data

Think of it as a lightweight Kafka alternative embedded inside Redis, ideal for microservices and serverless workflows.

---

#### Add Dependencies

Add the Spring Data Redis and Lettuce client dependencies to your `pom.xml`:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
<groupId>io.lettuce</groupId>
<artifactId>lettuce-core</artifactId>
</dependency>
```

Also configure Redis in `application.yml`:

```yml
spring:
redis:
host: localhost
port: 6379
```

---

#### Writing to a Redis Stream (Producer)

Use `RedisTemplate` or `StreamOperations` to add records to a stream.

```java
@Autowired
private StringRedisTemplate redisTemplate;

public void publishEvent(String streamKey, Map<String, String> eventData) {
redisTemplate.opsForStream()
.add(StreamRecords.mapBacked(eventData).withStreamKey(streamKey));
}
```

Example event:

```java
Map<String, String> data = Map.of(
"user", "alice",
"action", "login",
"timestamp", Instant.now().toString()
);
publishEvent("user.activity", data);
```

---

#### Reading from a Stream (Simple Consumer)

You can poll a Redis stream manually:

```java
List<MapRecord<String, Object, Object>> messages =
redisTemplate.opsForStream()
.range("user.activity", Range.unbounded());
for (MapRecord<String, Object, Object> msg : messages) {
System.out.println("Message: " + msg.getValue());
}
```

This reads all messages — useful for debugging or batch reads.

---

#### Using Consumer Groups for Scalability

To consume from a stream in a scalable and fault-tolerant way, use **consumer groups**.

```java
// Create the group if not exists
StreamOperations<String, Object, Object> ops = redisTemplate.opsForStream();
try {
ops.createGroup("user.activity", "activityGroup");
} catch (RedisSystemException e) {
// Group already exists
}
```

Now listen as a specific consumer:

```java
List<MapRecord<String, Object, Object>> messages = ops.read(
Consumer.from("activityGroup", "consumer-1"),
StreamReadOptions.empty().count(10).block(Duration.ofSeconds(5)),
StreamOffset.create("user.activity", ReadOffset.lastConsumed())
);
```

After processing, acknowledge:

```java
ops.acknowledge("user.activity", "activityGroup", messages.get(0).getId());
```

This guarantees **at-least-once delivery**.

---

#### Auto-Configure with Spring Data Redis StreamListener

Spring Data Redis offers support for listeners with `StreamListener`:

```java
@Component
public class UserActivityListener implements StreamListener<String, MapRecord<String, Object, Object>> {

    @Override
    public void onMessage(MapRecord<String, Object, Object> message) {
        System.out.println("Received: " + message.getValue());
        // process event...
    }
}
```

Enable via `StreamMessageListenerContainer`:

```java
@Bean
public StreamMessageListenerContainer<String, MapRecord<String, Object, Object>> listenerContainer(
RedisConnectionFactory connectionFactory,
UserActivityListener listener) {

    StreamMessageListenerContainer<String, MapRecord<String, Object, Object>> container =
        StreamMessageListenerContainer.create(connectionFactory,
            StreamMessageListenerContainerOptions.builder()
                .pollTimeout(Duration.ofMillis(100))
                .build());

    container.receive(Consumer.from("activityGroup", "consumer-1"),
        StreamOffset.create("user.activity", ReadOffset.lastConsumed()),
        listener);

    container.start();
    return container;
}
```

---

#### Retention, Trimming, and Replay

Redis Streams can automatically **trim old entries**:

```java
ops.trim("user.activity", 1000); // Keep last 1000 entries
```

Or configure max length:

```java
redisTemplate.opsForStream()
.add(StreamRecords.mapBacked(data).withStreamKey("user.activity"))
.withMaxLen(1000);
```

To **replay** unacknowledged messages:

```java
List<MapRecord<String, Object, Object>> pending =
ops.readPending("user.activity", Consumer.from("activityGroup", "consumer-1"), Range.unbounded(), 10);
```

---

#### Best Practices

- Use consumer groups for reliable processing
- Acknowledge messages after processing to avoid redelivery
- Set stream TTL or use trimming to prevent unbounded growth
- Use meaningful stream keys (`event:order:created`, etc.)
- Consider partitioning large streams across keys

---

#### Conclusion

Redis Streams and Spring Boot form a powerful duo for building **real-time, event-driven applications**. Whether you're tracking user behavior, processing system logs, or building microservices communication pipelines, Redis Streams offer an elegant and efficient solution.

By using **consumer groups**, **stream trimming**, and **Spring integrations**, you can deliver fast, reliable, and scalable event processing with minimal infrastructure.
