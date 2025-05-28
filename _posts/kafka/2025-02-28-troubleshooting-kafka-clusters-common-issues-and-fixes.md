---
layout: post
title: Troubleshooting Kafka Clusters Common Issues and Fixes
subtitle: Identify and resolve common Apache Kafka cluster problems with practical troubleshooting techniques
categories: Kafka
tags: [Kafka, Troubleshooting, Cluster Management, DevOps, Monitoring, Event Streaming, Apache Kafka]
excerpt: Learn how to diagnose and fix common Apache Kafka cluster issues, including broker failures, consumer lag, under-replicated partitions, and misconfigurations in real-time data pipelines.
---
Apache Kafka is a distributed, high-throughput messaging platform that powers many real-time applications and data pipelines. While Kafka is robust, maintaining a healthy Kafka cluster can be challenging, especially at scale.

This guide provides a practical approach to **troubleshooting common Kafka cluster issues**, covering symptoms, root causes, and actionable fixes. Whether you're dealing with **broker crashes**, **consumer lag**, or **replication failures**, this post will help you keep your Kafka system running smoothly.

---

#### 1. Broker Is Down or Unresponsive

**Symptoms:**
- Producer errors: `Connection refused`
- Consumer failures: `Broker not available`
- Missing metrics in monitoring tools

**Causes:**
- JVM crashes or out-of-memory errors
- Misconfigured listeners or advertised.listeners
- Port conflicts or firewall blocks

**Fix:**
- Check broker logs (e.g., `/var/log/kafka/server.log`) for errors
- Validate Zookeeper connectivity
- Confirm correct listener and advertised.listener settings
- Restart broker and monitor with `kafka-broker-api-versions.sh`

---

#### 2. Under-Replicated Partitions

**Symptoms:**
- `UnderReplicatedPartitions` metric > 0
- Alerts from monitoring tools
- High ISR shrink/expand rate

**Causes:**
- Slow broker recovery
- Network latency between brokers
- Disk IO bottlenecks

**Fix:**
- Use CLI:  
  ```bash
  kafka-topics.sh --describe --topic your-topic --bootstrap-server broker:9092
  ```
- Check for broker logs indicating replication delay
- Tune `replica.fetch.max.bytes` and `replica.fetch.wait.max.ms`
- Restart slow brokers gracefully

---

#### 3. Consumer Lag Growing

**Symptoms:**
- Lag metrics rising (check with Burrow or Prometheus)
- Consumer apps falling behind
- Out-of-order processing or late events

**Causes:**
- Slow consumers
- Not enough partitions for parallelism
- Long GC pauses or insufficient memory

**Fix:**
- Scale consumer group horizontally
- Increase topic partitions (use Admin API cautiously)
- Tune consumer configs:
  ```
  max.poll.records=500
  fetch.max.bytes=52428800
  ```
- Profile consumer app for bottlenecks

---

#### 4. Message Loss or Duplicate Delivery

**Symptoms:**
- Events missing from sinks (e.g., DB, S3)
- Duplicate writes to external systems
- Unexpected behavior in streaming applications

**Causes:**
- Producers not using idempotent writes
- Consumers not committing offsets properly
- Non-transactional sink operations

**Fix:**
- Enable idempotent producer:
  ```
  enable.idempotence=true
  ```
- Use `enable.auto.commit=false` and manually commit after successful processing
- For exactly-once, use Kafka Transactions with proper transaction boundaries

---

#### 5. ZooKeeper or Controller Election Issues

**Symptoms:**
- Brokers not joining the cluster
- No active controller elected
- Frequent controller failovers

**Causes:**
- Network issues between brokers and ZooKeeper
- Clock skew or jitter
- High ZooKeeper latency

**Fix:**
- Ensure ZooKeeper ensemble is healthy (`zkServer.sh status`)
- Reduce number of partitions to limit controller workload
- Use `kafka-metadata-quorum.sh` (Kafka KRaft mode) if on Kafka ≥ 3.3

---

#### 6. SSL/TLS or Authentication Failures

**Symptoms:**
- TLS handshake errors in logs
- Clients fail to connect securely
- Kafka Connect or MirrorMaker showing auth errors

**Causes:**
- Invalid or expired certificates
- Incorrect keystore/truststore configs
- SASL or ACL misconfigurations

**Fix:**
- Validate keystore and truststore paths
- Ensure all brokers share the correct CA chain
- Check client logs for:
  ```
  javax.net.ssl.SSLHandshakeException
  ```
- Rotate expiring certs before renewal deadlines

---

#### 7. Disk Usage or Log Retention Problems

**Symptoms:**
- Disk space alerts
- Kafka log cleaner stalling
- Topics not respecting retention policies

**Causes:**
- Retention period too long
- Log compaction misconfigured
- Partitions not getting deleted

**Fix:**
- Set retention policy in topic config:
  ```
  kafka-configs.sh --alter --topic your-topic \
  --add-config retention.ms=604800000 \
  --bootstrap-server broker:9092
  ```
- Enable log cleaner for compacted topics
- Set quotas for producer to prevent overproduction

---

#### 8. Partition Skew or Hot Partitions

**Symptoms:**
- Uneven broker load
- High throughput or CPU on a subset of brokers
- Consumer lag only on certain partitions

**Causes:**
- Poor partition key design (low cardinality)
- Uneven message distribution from producers

**Fix:**
- Use high-cardinality partition keys
- Monitor partition distribution:
  ```
  kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list broker:9092 --topic your-topic --time -1
  ```
- Repartition topics with Kafka Streams or Flink if needed

---

#### Tools for Kafka Troubleshooting

- **Kafka CLI Tools**: `kafka-topics.sh`, `kafka-consumer-groups.sh`, `kafka-configs.sh`
- **Monitoring**: Prometheus, Grafana, Burrow, Confluent Control Center
- **Logging**: Check logs under `/var/log/kafka/`
- **JMX Exporter**: For broker metrics
- **Cruise Control**: For broker rebalancing and optimization

---

#### Conclusion

Apache Kafka is a production-grade system, but it’s not immune to misconfigurations, network failures, and performance bottlenecks. By learning how to **diagnose and fix common issues**, you can ensure **maximum uptime**, **data integrity**, and **operational efficiency** in your Kafka-based systems.

Always combine **proactive monitoring**, **best practices**, and **automated alerts** to detect issues early and resolve them before they impact users.
