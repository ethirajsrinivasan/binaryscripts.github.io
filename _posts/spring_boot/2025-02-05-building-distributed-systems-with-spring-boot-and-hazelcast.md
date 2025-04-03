---
layout: post
title: Building Distributed Systems with Spring Boot and Hazelcast
subtitle: Design scalable and resilient distributed systems using Spring Boot and Hazelcast's in-memory data grid
categories: Spring Boot
tags: [Java, Spring Boot, Hazelcast, Distributed Systems, Caching, Clustering, Microservices]
excerpt: Learn how to build scalable distributed systems using Spring Boot and Hazelcast. Explore clustering, in-memory caching, pub-sub messaging, and distributed data structures to build resilient microservices.
---



Modern applications must scale horizontally and provide **high availability**, especially in cloud-native and microservices-based environments. Building such distributed systems often introduces challenges around **state sharing**, **data consistency**, and **coordination**.

This is where **Hazelcast**, an in-memory data grid (IMDG), shines. It enables **data sharing, caching, clustering**, and **distributed computing** — all integrated seamlessly with **Spring Boot**.

In this guide, we’ll cover how to use Hazelcast with Spring Boot to build fault-tolerant and scalable distributed systems.

---

#### What Is Hazelcast?

Hazelcast is a **distributed computing platform** that offers:
- In-memory storage (data grid)
- Clustering and node discovery
- Distributed caching and maps
- Distributed locks and semaphores
- Pub-sub messaging
- Entry processors and distributed executors

Hazelcast runs embedded in your application or in client-server mode, making it versatile for cloud-native deployments.

---

#### Adding Hazelcast to a Spring Boot Project

Add the following dependency to your `pom.xml`:

```xml
<dependency>
<groupId>com.hazelcast</groupId>
<artifactId>hazelcast</artifactId>
<version>5.3.6</version>
</dependency>
<dependency>
<groupId>com.hazelcast</groupId>
<artifactId>hazelcast-spring</artifactId>
</dependency>
```

Spring Boot auto-configures Hazelcast if it finds a `hazelcast.xml` or `hazelcast.yaml` in the classpath.

---

#### Cluster Discovery and Configuration

Start with a simple `hazelcast.yaml` in `src/main/resources/`:

```yaml
hazelcast:
cluster-name: spring-cluster
network:
join:
multicast:
enabled: true
tcp-ip:
enabled: false
```

When multiple Spring Boot apps run in the same network, they form a cluster automatically using multicast discovery.

You can also configure Hazelcast programmatically:

```java
@Bean
public Config hazelcastConfig() {
Config config = new Config();
config.setClusterName("spring-cluster");
config.getNetworkConfig().getJoin().getMulticastConfig().setEnabled(true);
return config;
}
```

---

#### Using Hazelcast as a Distributed Cache

Hazelcast integrates with Spring’s `@Cacheable` abstraction.

Enable caching:

```java
@SpringBootApplication
@EnableCaching
public class Application {}
```

Then annotate your service methods:

```java
@Cacheable(value = "products", key = "#id")
public Product getProductById(Long id) {
return repository.findById(id).orElseThrow();
}
```

Hazelcast will store results in a **distributed map**, available across all nodes in the cluster.

---

#### Distributed Maps and Data Structures

Hazelcast provides distributed implementations of common Java collections:

```java
@Autowired
private HazelcastInstance hazelcastInstance;

public void storeData() {
IMap<String, String> map = hazelcastInstance.getMap("config-data");
map.put("feature.enabled", "true");
}
```

Other distributed structures:
- `IQueue` – distributed queue
- `ISemaphore` – distributed semaphore
- `ILock` – distributed locking
- `ITopic` – pub-sub messaging

---

#### Pub/Sub Messaging with Hazelcast Topics

You can build **event-driven systems** using Hazelcast’s distributed topic:

```java
ITopic<String> topic = hazelcastInstance.getTopic("user-events");

topic.addMessageListener(message -> {
System.out.println("Received: " + message.getMessageObject());
});

topic.publish("User registered");
```

This allows microservices to communicate asynchronously without external brokers.

---

#### Using Hazelcast for Session Replication

In stateful applications, Hazelcast can replicate HTTP sessions across nodes:

Add dependency:

```xml
<dependency>
<groupId>com.hazelcast</groupId>
<artifactId>hazelcast-wm</artifactId>
</dependency>
```

Configure session sharing in `application.properties`:

```properties
server.servlet.session.timeout=30m
spring.session.store-type=hazelcast
```

This ensures users don’t lose their session during failovers or scaling events.

---

#### Health Checks and Cluster Monitoring

Hazelcast provides a REST endpoint and management center (available as a Docker container or standalone) to monitor nodes, maps, memory usage, and cluster health.

Start Hazelcast Management Center via Docker:

```bash
docker run -p 8080:8080 hazelcast/management-center
```

Access it at `http://localhost:8080` to view cluster topology and metrics.

---

#### Best Practices

- Use **asynchronous APIs** (`IExecutorService`, non-blocking maps) for performance
- Prefer **Hazelcast client-server mode** in large clusters
- Configure **data eviction** and **persistence** for large or long-lived data
- Combine with **Spring Cloud** for service discovery
- Secure Hazelcast clusters with TLS and authentication

---

#### Conclusion

Spring Boot and Hazelcast offer a powerful stack for building **distributed, resilient, and high-performance applications**. Whether you're caching frequently accessed data, managing sessions, or implementing pub-sub messaging, Hazelcast gives you the primitives needed to scale out your architecture safely.

As distributed systems become the norm, mastering Hazelcast with Spring Boot puts you ahead in building systems that are both **robust and cloud-ready**.
