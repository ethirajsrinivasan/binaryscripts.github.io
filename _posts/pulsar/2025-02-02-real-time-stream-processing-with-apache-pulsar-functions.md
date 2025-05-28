---
layout: post
title: Real Time Stream Processing with Apache Pulsar Functions
subtitle: Build lightweight and scalable real-time data pipelines using Apache Pulsar Functions
categories: Pulsar
tags: [Pulsar, Stream Processing, Real-Time, Pulsar Functions, Serverless, Event-Driven, Big Data]
excerpt: Learn how to build real-time stream processing pipelines using Apache Pulsar Functions. Explore deployment modes, processing logic, scaling strategies, and integration with modern data platforms.
---
As modern applications become more event-driven, there's a growing need to process data in real time — from filtering logs to enriching messages and triggering alerts. **Apache Pulsar Functions** offer a lightweight, serverless compute framework for running real-time stream processing logic directly within the Pulsar messaging system.

In this guide, we’ll explore how to implement **real-time stream processing using Pulsar Functions**, including deployment models, scaling strategies, and best practices for efficient pipeline development.

---

#### What are Pulsar Functions?

**Pulsar Functions** are user-defined, event-driven functions that run inside the Apache Pulsar framework to process messages as they arrive.

Key features:
- **Low-latency stream processing**
- **Built-in scaling and fault tolerance**
- Supports **Java, Python, and Go**
- Seamlessly integrates with **Pulsar topics**
- No need for external compute frameworks like Flink or Spark

---

#### When to Use Pulsar Functions

Use Pulsar Functions when you need to:
- Enrich or transform messages in real-time
- Filter and route events across topics
- Perform lightweight aggregations
- Trigger alerts or downstream systems
- Implement custom pipelines without managing external stream processors

---

#### Pulsar Function Architecture

```
[Producer]
↓
[Input Topic]
↓
[Pulsar Function (Transform/Enrich/Filter)]
↓
[Output Topic(s) or External System]
```

Each function:
- Subscribes to one or more input topics
- Processes messages (optionally stateful)
- Writes results to output topics or external sinks

---

#### Example: Simple Transformation Function (Python)

```python
def process_message(input, context):
# Add metadata to each message
return f"{input} | processed at {context.get_message_id()}"

# Save as transform.py
```

Deploy using:

```bash
pulsar-admin functions create \
--name enrich-logs \
--runtime python \
--py transform.py \
--input-topic raw-logs \
--output-topic enriched-logs \
--tenant public \
--namespace default
```

---

#### Stateful Processing

Pulsar Functions support **stateful processing** using built-in state stores.

Example (Java):

```java
public String process(String input, Context context) {
int count = context.getState("count") == null ? 0 : Integer.parseInt(context.getState("count"));
count += 1;
context.putState("count", String.valueOf(count));
return input + " | msg #" + count;
}
```

State is persisted to BookKeeper and can be queried externally.

---

#### Deploying Functions

Deployment options:
- **CLI**: pulsar-admin or `pulsar-function-local-run`
- **REST API**: For programmatic management
- **Kubernetes**: With Pulsar Operator or Helm charts
- **Functions Worker**: Runs on brokers or standalone workers

You can also package functions as **JARs**, **Python scripts**, or **Docker images**.

---

#### Scaling Pulsar Functions

Configure parallelism with:

```
--parallelism 3
```

Pulsar will create 3 instances of the function and spread them across available workers.

Use **key-based routing** for sticky assignment:

```python
context.get_output_topic_producer("output").send_async(key, value)
```

---

#### Integrating with External Systems

Pulsar Functions can call external APIs or emit to other systems:

```python
import requests

def process(input, context):
requests.post("https://api.example.com/event", json={"msg": input})
return input
```

Use this for:
- Triggering webhooks
- Posting alerts
- Sending data to monitoring systems

---

#### Monitoring and Observability

Monitor functions with:

- **pulsar-admin functions stats**
- **Prometheus metrics** from function workers
- **Function logs** via CLI or broker logs

Metrics include:
- Processed message count
- Failures
- Latency
- Resource usage

---

#### Best Practices

✅ Keep logic simple and lightweight  
✅ Use stateful features sparingly to reduce overhead  
✅ Handle errors and log exceptions clearly  
✅ Use environment variables or configs for parameters  
✅ Version and test functions before deploying to production  
✅ Monitor performance and scale based on throughput

---

#### Conclusion

Apache Pulsar Functions offer a powerful yet lightweight way to build **real-time stream processing applications** directly inside your messaging infrastructure. By reducing operational complexity and allowing inline transformation, Pulsar Functions enable fast, scalable, and efficient data pipelines for modern event-driven architectures.

Whether you're enriching messages, routing events, or powering alerts — Pulsar Functions give you the tools to process streaming data in real time, with minimal overhead.
