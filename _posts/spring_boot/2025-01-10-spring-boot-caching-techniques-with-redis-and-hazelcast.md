---
layout: post
title: Spring Boot Caching Techniques with Redis and Hazelcast
subtitle: Enhance Java application performance with Spring Boot caching using Redis and Hazelcast
categories: SpringBoot
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: [Java, Spring Boot, Caching, Redis, Hazelcast, Performance, In-Memory]
excerpt: Learn how to implement powerful caching strategies in Spring Boot using Redis and Hazelcast. Improve performance, reduce database load, and build responsive Java applications.
---
Caching is a vital performance optimization in modern web applications. It helps reduce response times, offload expensive computations, and alleviate pressure on your databases. Spring Boot makes caching straightforward and extensible — and when combined with distributed in-memory data stores like **Redis** and **Hazelcast**, you gain **speed, scalability, and resilience**.

In this guide, we explore caching strategies using **Spring Boot**, **Redis**, and **Hazelcast**. You’ll learn when and how to cache, choose the right provider, and avoid common pitfalls.

---

#### Why Use Caching?

Caching improves performance by storing frequently accessed data in memory instead of fetching it repeatedly from slower sources like databases or APIs.

**Benefits of caching in Java apps:**
- Faster response time
- Lower latency for critical endpoints
- Reduced database load
- Better scalability under high load

---

#### Enabling Caching in Spring Boot

Start by enabling caching support in your application:

```java
@SpringBootApplication
@EnableCaching
public class Application { }
```

Spring Boot supports multiple caching providers, including:
- Simple in-memory cache (default)
- Redis
- Hazelcast
- Ehcache
- JCache (JSR-107)

---

#### Using Cache Annotations

The Spring Cache abstraction provides a simple annotation-based API:

```java
@Cacheable("users")
public User getUserById(Long id) {
return userRepository.findById(id).orElseThrow();
}
```

Other useful annotations:
- `@CachePut` – updates the cache
- `@CacheEvict` – removes data from the cache
- `@Caching` – group multiple operations

```java
@CacheEvict(value = "users", key = "#id")
public void deleteUser(Long id) {
userRepository.deleteById(id);
}
```

---

#### Integrating Redis as Cache Provider

Redis is a fast, persistent, key-value store often used for caching.

**Add dependency:**

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**Configure Redis in `application.yml`:**

```yml
spring:
cache:
type: redis
redis:
host: localhost
port: 6379
```

Spring Boot automatically uses Redis as the cache backend.

Customize serialization (e.g., use JSON instead of JDK serialization):

```java
@Bean
public RedisCacheConfiguration cacheConfiguration() {
return RedisCacheConfiguration.defaultCacheConfig()
.serializeValuesWith(SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
}
```

---

#### Integrating Hazelcast as a Distributed Cache

Hazelcast is a distributed in-memory computing platform with built-in clustering and auto-discovery.

**Add Hazelcast dependency:**

```xml
<dependency>
<groupId>com.hazelcast</groupId>
<artifactId>hazelcast-spring</artifactId>
</dependency>
```

**Set Spring to use Hazelcast:**

```yml
spring:
cache:
type: hazelcast
```

**Configure Hazelcast:**

```java
@Bean
public Config hazelcastConfig() {
Config config = new Config();
config.setInstanceName("hazelcast-instance")
.addMapConfig(new MapConfig()
.setName("users")
.setTimeToLiveSeconds(600));
return config;
}
```

Hazelcast supports clustering, so cached data is shared across nodes.

---

#### Choosing Between Redis and Hazelcast

| Feature         | Redis                          | Hazelcast                      |
|------------------|--------------------------------|--------------------------------|
| Persistence      | Optional RDB/AOF              | In-memory only (unless IMDG)  |
| Clustering       | Master-replica + Sentinel     | Peer-to-peer                   |
| TTL Support      | Built-in per-key TTL          | Per-map TTL                    |
| Serialization    | Configurable, supports JSON   | Uses Java serialization by default |
| Use Case         | Web caching, queues, sessions | Distributed data grids, sessions |

Choose **Redis** when:
- You want fast, persistent, and standalone caching
- You're already using Redis for sessions, queues, etc.

Choose **Hazelcast** when:
- You need cluster-wide caching with Java-native integration
- You want simple discovery between Spring Boot nodes

---

#### Expiration and Eviction Policies

Both Redis and Hazelcast support TTL and eviction settings.

- Use **short TTLs** for dynamic data
- Set **max size** to avoid memory overuse
- Evict least-recently-used (LRU) entries when full

```java
@Cacheable(value = "users", key = "#id", unless = "#result == null")
public User getCachedUser(Long id) { ... }
```

---

#### Caching Best Practices

- Use **cache abstraction**, not direct Redis APIs, for flexibility
- Avoid caching mutable or sensitive data
- Test cache invalidation paths
- Log and monitor cache hit/miss ratios
- Combine with Spring AOP for cross-cutting cache behavior

---

#### Conclusion

Caching is one of the most impactful techniques for boosting application performance. By leveraging Spring Boot with **Redis** and **Hazelcast**, you can build caching layers that scale with your app.

Whether you choose Redis for simplicity and speed or Hazelcast for distributed clustering, Spring Boot makes integration easy and extensible — so you can focus on delivering fast, responsive user experiences.
