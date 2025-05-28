---
layout: post
title: Pulsar Schema Registry Managing Schema Evolution in Real Time Streams
subtitle: Learn how Apache Pulsar handles schema evolution and compatibility for reliable, real-time data pipelines
categories: Pulsar
tags: [Pulsar, Schema Registry, Schema Evolution, Real-Time Streams, Apache Pulsar, Event Streaming, Data Compatibility]
excerpt: Explore how Apache Pulsar's built-in schema registry enables schema evolution and enforcement in real-time event streams. Learn schema types, compatibility settings, and best practices for safe data pipeline evolution.
---
In modern event-driven systems, maintaining **data consistency** is critical — especially when producers and consumers evolve independently. Schema changes can lead to pipeline breakage, data loss, or application errors if not handled carefully.

**Apache Pulsar** solves this with a **built-in schema registry**, providing native support for **schema enforcement**, **evolution**, and **type safety** — all without relying on an external system like in Kafka.

In this post, you'll learn how Pulsar's **schema registry** works, how to configure **schema compatibility policies**, and best practices for managing **schema evolution in real-time streams**.

---

#### What is Pulsar Schema Registry?

Apache Pulsar's schema registry is:

- **Built into the Pulsar broker** (no separate deployment)
- Manages **schemas per topic**
- Tracks **schema versions** and ensures **backward/forward compatibility**
- Supports multiple formats: **Avro, JSON, Protobuf, String, Bytes**

It allows producers and consumers to agree on **message formats** and automatically validates data against the expected schema.

---

#### Why Use a Schema Registry?

- **Strong typing**: Prevents accidental publishing of incompatible data
- **Compatibility enforcement**: Ensures downstream consumers don't break with schema changes
- **Introspection**: View and manage schema versions directly
- **Tooling**: Enables easier integration with stream processors and analytics tools

---

#### Supported Schema Types

Pulsar supports several schema types:

| Schema Type  | Description                                  |
|--------------|----------------------------------------------|
| `AVRO`       | Compact, widely used with schema evolution   |
| `PROTOBUF`   | Efficient, strongly typed                    |
| `JSON`       | Human-readable, less efficient               |
| `STRING`     | Simple UTF-8 encoding                        |
| `BYTES`      | No schema enforcement (raw)                  |
| `KEY_VALUE`  | Schema for key-value pairs (partitioned topics) |

---

#### Defining Schemas in Producers

Example: Java Avro schema

```java
@AvroSchema
public class UserEvent {
public String userId;
public long timestamp;
public String action;
}

Producer<UserEvent> producer = pulsarClient.newProducer(Schema.AVRO(UserEvent.class))
.topic("persistent://public/default/user-events")
.create();
```

Pulsar automatically registers the schema and validates future messages against it.

---

#### Schema Compatibility Strategies

When schema evolves, Pulsar validates compatibility using:

| Compatibility Level | Description                              |
|---------------------|------------------------------------------|
| `FULL`              | Backward and forward compatible          |
| `BACKWARD`          | New schema can read old data             |
| `FORWARD`           | Old consumers can read new data          |
| `ALWAYS_COMPATIBLE` | No restrictions                          |
| `NONE`              | No compatibility checks (risky)          |

Set compatibility using CLI:

```bash
pulsar-admin schemas compatibility-strategy set \
--compatibility BACKWARD \
public/default/user-events
```

---

#### Viewing and Managing Schemas

List schema versions:

```bash
pulsar-admin schemas get --topic persistent://public/default/user-events
```

Delete a schema version:

```bash
pulsar-admin schemas delete \
--topic persistent://public/default/user-events \
--version <version>
```

Note: Deleting schemas can break consumers — use with caution.

---

#### Schema Evolution Example

Initial schema:

```json
{
"type": "record",
"name": "UserEvent",
"fields": [
{ "name": "userId", "type": "string" },
{ "name": "timestamp", "type": "long" }
]
}
```

Evolved schema (adds optional field):

```json
{
"type": "record",
"name": "UserEvent",
"fields": [
{ "name": "userId", "type": "string" },
{ "name": "timestamp", "type": "long" },
{ "name": "action", "type": ["null", "string"], "default": null }
]
}
```

This is **backward-compatible**, as old consumers can ignore the new field.

---

#### Best Practices for Managing Schema Evolution

- **Use Avro or Protobuf** for production — they provide rich type and compatibility support
- Always specify a **default value** for new fields
- Avoid removing or renaming existing fields
- Validate schema changes in staging before production deployment
- Keep compatibility set to `BACKWARD` for safety
- Monitor schema registry changes via CLI or REST APIs

---

#### Integration with Stream Processors

Pulsar schemas work out of the box with:

- **Flink Pulsar Source/Sink** (schema-aware)
- **Spark Structured Streaming** via DataFrames
- **Pulsar Functions** with typed inputs/outputs
- **ksqlDB** and external processors using Avro over Kafka compatibility layer

---

#### Conclusion

Schema evolution is inevitable in dynamic data systems. Apache Pulsar makes it seamless and safe by offering a **native schema registry**, **compatibility strategies**, and **version control** for real-time data streams.

By using Pulsar's schema capabilities, you can **future-proof your data pipelines**, enforce **data integrity**, and scale confidently as your applications evolve.
