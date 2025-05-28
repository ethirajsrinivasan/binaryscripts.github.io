---
layout: post
title: Integrating Apache Pulsar with NoSQL Databases for Real Time Data Pipelines
subtitle: Build scalable and low latency pipelines by connecting Apache Pulsar with NoSQL databases like MongoDB, Cassandra and DynamoDB
categories: Pulsar
tags: [Apache Pulsar, NoSQL, MongoDB, Cassandra, DynamoDB, Streaming, Real-Time Data, Event Driven Architecture, Big Data, Data Pipelines]
excerpt: Learn how to connect Apache Pulsar with popular NoSQL databases to build real-time data pipelines. This post explores integration patterns, connectors, schema handling, and best practices for low-latency ingestion and analytics.
---
In the era of *real-time applications*, data must flow instantly from producers to analytics engines and storage systems. Apache Pulsar, with its *event-driven architecture*, is built to handle high-throughput streaming data. When combined with **NoSQL databases** like **MongoDB**, **Apache Cassandra**, or **Amazon DynamoDB**, Pulsar can power robust, scalable, and highly responsive pipelines.

This blog dives into *technical best practices* for integrating Pulsar with NoSQL databases. We'll explore **connectors, change data capture (CDC)**, schema handling, and real-world examples that help bridge messaging with flexible, distributed data stores.

---

#### Why Combine Pulsar and NoSQL

**Apache Pulsar** excels in decoupling systems through asynchronous messaging, while **NoSQL databases** provide schema-less, horizontally scalable storage with low-latency reads/writes. Together, they offer:

- Real-time ingestion from microservices, IoT, or logs
- Scalable persistence of semi-structured documents or key-value pairs
- Flexible querying and analytics for downstream consumers
- Event replay capabilities for rebuilding states or troubleshooting

---

#### Integration Architecture Overview

A typical pipeline involves:

```
[ Producers ] → [ Apache Pulsar Topics ] → [ Pulsar IO Connector ] → [ NoSQL Database ]
```

- **Producers**: Microservices, Kafka Bridges, IoT sensors, etc.
- **Topics**: Pulsar topics where events are published
- **Pulsar IO Connectors**: Prebuilt sink connectors to push data to NoSQL systems
- **NoSQL Store**: MongoDB, Cassandra, DynamoDB, etc.

---

#### Supported NoSQL Connectors

Apache Pulsar’s **Pulsar IO framework** offers out-of-the-box connectors for many NoSQL databases.

| NoSQL Database | Connector Type | Notes                                |
|----------------|----------------|--------------------------------------|
| MongoDB        | Sink           | Inserts or updates documents         |
| Cassandra      | Sink           | Writes events to CQL tables          |
| DynamoDB       | Sink           | Uses AWS SDK (via AWS Pulsar Sink)   |
| Couchbase      | Sink           | Compatible with JSON documents       |

To install a connector:

```bash
bin/pulsar-admin connectors create \
--archive connectors/pulsar-io-mongo-2.10.4.nar \
--name mongo-sink \
--sink-config-file mongo-sink-config.json
```

---

#### MongoDB Integration Example

**Step 1**: Configure `mongo-sink-config.json`

```json
{
"mongoUri": "mongodb://user:pass@mongo-host:27017",
"database": "realtime_db",
"collection": "events",
"batchSize": 100,
"batchTimeMs": 500
}
```

**Step 2**: Launch the sink

```bash
bin/pulsar-admin sinks create \
--name pulsar-mongo \
--sink-type mongo \
--inputs my-topic \
--sink-config-file mongo-sink-config.json \
--tenant public \
--namespace default
```

*Note: Messages should be in JSON format to match MongoDB’s document model.*

---

#### Cassandra Integration Example

Create a compatible schema:

```sql
CREATE TABLE sensor_data (
sensor_id text PRIMARY KEY,
timestamp bigint,
temperature float,
humidity float
);
```

Configure the sink:

```json
{
"roots": "127.0.0.1",
"keyspace": "pulsar_ks",
"columnFamily": "sensor_data",
"username": "cassandra",
"password": "cassandra"
}
```

---

#### DynamoDB Integration Example

Example config using the AWS Pulsar connector:

```json
{
"awsRegion": "us-east-1",
"awsAccessKeyId": "YOUR_KEY",
"awsSecretAccessKey": "YOUR_SECRET",
"tableName": "IoTEvents"
}
```

Deploy the sink:

```bash
bin/pulsar-admin sinks create \
--archive pulsar-io-aws-dynamodb.nar \
--name dynamo-sink \
--sink-config-file dynamodb-config.json \
--inputs iot-topic
```

---

#### Schema and Compatibility Considerations

Use Pulsar’s schema registry for strong schema enforcement:

```java
Schema<UserEvent> schema = Schema.JSON(UserEvent.class);
Producer<UserEvent> producer = client.newProducer(schema)
.topic("persistent://public/default/user-events")
.create();
```

**Tips**:
- Use `JSONSchema` or `AvroSchema` for compatibility
- Enable forward/backward schema versioning
- Align message structure with NoSQL document/table models

---

#### Real-Time Analytics Use Case

Example: Building a **real-time dashboard** using MongoDB:

1. Backend services publish events to Pulsar
2. Pulsar Mongo Sink stores them in a MongoDB collection
3. A frontend dashboard queries this collection for visualization
4. Optional: Pulsar Functions can enrich data before storage

Ideal for: *e-commerce analytics*, *IoT monitoring*, *real-time financial insights*.

---

#### Performance and Tuning Tips

- Enable **batch writes** in the sink configuration
- Use **partitioned topics** for parallelism
- Align Pulsar **ack timeouts** with sink write latencies
- Ensure NoSQL storage scaling matches throughput

---

#### Security Considerations

- Enable **TLS encryption** for Pulsar and DB connections
- Use **JWT authentication** and **role-based access control**
- Store secrets in **Kubernetes Secrets** or external vaults
- Restrict sink permissions by **namespace scope**

---

#### Conclusion

Apache Pulsar’s built-in support for NoSQL connectors and schema enforcement makes it an ideal platform for *building real-time data pipelines*. Whether you're logging IoT telemetry to MongoDB, persisting user actions in Cassandra, or ingesting retail events into DynamoDB, Pulsar enables *scalable, low-latency, and flexible* event delivery.

For modern, distributed architectures where **speed, scalability, and simplicity** matter, Pulsar + NoSQL is a proven, production-grade combination.

---
