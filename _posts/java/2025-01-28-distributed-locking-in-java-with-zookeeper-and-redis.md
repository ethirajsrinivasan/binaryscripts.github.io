---
layout: post
title: Distributed Locking in Java with Zookeeper and Redis
subtitle: Ensure data consistency in distributed Java applications using Zookeeper and Redis locks
categories: Java
tags: [Java, Zookeeper, Redis, Distributed Systems, Locking, Concurrency]
excerpt: Learn how to implement distributed locking in Java using Zookeeper and Redis. Prevent race conditions and ensure safe access to shared resources in distributed applications.
---



In distributed systems, multiple application nodes often compete to access shared resources — such as databases, queues, or configuration files. Without a proper locking mechanism, this can lead to **race conditions**, **data corruption**, and **inconsistent states**.

This post explores how to implement **distributed locking** in Java using **Apache Zookeeper** and **Redis**, two widely-used tools that offer reliable coordination and synchronization primitives.

---

#### Why Distributed Locks Are Needed

Consider a scheduled job running on multiple JVMs behind a load balancer. If each node tries to update the same database table at the same time, it can cause duplicate operations or conflict errors.

Distributed locks ensure that:
- Only one node can perform a critical operation at a time
- Other nodes wait or back off
- The system remains consistent and fault-tolerant

---

#### Implementing Distributed Locks with Zookeeper

Zookeeper is a consistent, fault-tolerant coordination service used for leader election, configuration management, and locking.

You can use the **Apache Curator** framework to simplify Zookeeper-based locks.

Add the Maven dependency:

```xml
<dependency>
<groupId>org.apache.curator</groupId>
<artifactId>curator-recipes</artifactId>
<version>5.5.0</version>
</dependency>
```

Create and use an `InterProcessMutex`:

```java
CuratorFramework client = CuratorFrameworkFactory.newClient(
"localhost:2181", new ExponentialBackoffRetry(1000, 3));
client.start();

InterProcessMutex lock = new InterProcessMutex(client, "/locks/my-lock");

try {
if (lock.acquire(5, TimeUnit.SECONDS)) {
// critical section
doWork();
}
} finally {
lock.release();
}
```

This ensures mutual exclusion across multiple JVMs.

---

#### Redis-Based Distributed Locks

Redis offers a lightweight locking mechanism using the `SET key value NX PX` command, often combined with expiration to prevent deadlocks.

Libraries like **Redisson** or **Lettuce** make this easy in Java.

Example with Redisson:

```xml
<dependency>
<groupId>org.redisson</groupId>
<artifactId>redisson</artifactId>
<version>3.18.0</version>
</dependency>
```

```java
Config config = new Config();
config.useSingleServer().setAddress("redis://127.0.0.1:6379");

RedissonClient redisson = Redisson.create(config);
RLock lock = redisson.getLock("myLock");

lock.lock();
try {
// critical section
performTask();
} finally {
lock.unlock();
}
```

Redis locks are fast, but be aware of network partitions and expiration races.

---

#### RedLock Algorithm (Optional)

To increase safety in distributed environments, use **RedLock** — an algorithm proposed by Redis' creator, which uses multiple Redis nodes to acquire locks with quorum-based validation.

Redisson supports RedLock out of the box, but it’s recommended for advanced, high-availability setups only.

---

#### Handling Failures and Timeouts

Always guard against:
- Lock holder crashes without releasing the lock
- Network timeouts
- Retry storms

**Tips:**
- Set lock expiration (`TTL`) to avoid orphaned locks
- Implement exponential backoff on retries
- Use unique lock IDs to prevent accidental unlocks by other clients

---

#### Choosing Between Zookeeper and Redis

| Feature       | Zookeeper                  | Redis                     |
|---------------|----------------------------|---------------------------|
| Consistency   | Strong (CP in CAP)         | Eventual (AP in CAP)      |
| Use Case      | Long-term locks, election  | Fast, short-lived locks   |
| Complexity    | Higher                     | Lower                     |
| Failure Handling | Built-in retries        | Must handle manually      |

Use Zookeeper for critical, long-running operations. Use Redis for high-performance, lightweight locks.

---

#### Conclusion

Distributed locking is essential for ensuring safe and consistent access to shared resources across JVMs or services. Both **Zookeeper** and **Redis** offer robust ways to implement distributed locks in Java, each with its own trade-offs.

By choosing the right tool and following locking best practices, you can design reliable and resilient distributed systems.
