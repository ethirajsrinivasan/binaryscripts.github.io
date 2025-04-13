---
layout: post
title: "Real-Time Data Processing with Flask and Redis Streams"
subtitle: "Learn how to process real-time data efficiently using Flask and Redis Streams"
categories: Flask
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: ["Flask", "Redis Streams", "Real-Time Data", "Event-Driven Architecture", "Streaming", "Python", "Redis"]
excerpt: "Discover how to integrate Redis Streams with Flask for real-time data processing. Learn how to publish, consume, and process streaming data efficiently."
---
In today’s data-driven world, **real-time data processing** is crucial for applications that require instant event handling, analytics, and notifications. **Redis Streams** provides an **efficient event-driven architecture**, enabling applications to process and analyze continuous data streams.

In this guide, we will explore **how to integrate Redis Streams with Flask** to **publish, consume, and process real-time data** efficiently.

## What is Redis Streams?

Redis Streams is a **message broker and event-processing system** that allows:

- **Real-time data ingestion**
- **Message persistence**
- **Consumer groups for distributed processing**
- **Automatic message acknowledgment**

Redis Streams is ideal for **log processing, event-driven applications, and real-time analytics**.

## Setting Up Flask and Redis

### Installing Dependencies

First, install **Flask, Redis, and Redis-Py** using **pip**:

```sh
pip install flask redis
```

Ensure Redis is running on your system:

```sh
redis-server
```

### Configuring Redis in Flask

Create a Flask app and configure a Redis connection:

```python
from flask import Flask, request, jsonify
import redis

app = Flask(__name__)
redis_client = redis.Redis(host='localhost', port=6379, decode_responses=True)
STREAM_NAME = "events"
```

Now, we are ready to **publish and consume real-time events**.

## Publishing Events to Redis Streams

To **push events into Redis Streams**, define an API endpoint:

```python
@app.route('/publish', methods=['POST'])
def publish_event():
data = request.get_json()
event_id = redis_client.xadd(STREAM_NAME, data)
return jsonify({"event_id": event_id, "status": "Event published"}), 201
```

This will **store incoming events in Redis Streams**, making them available for consumers.

## Consuming Events from Redis Streams

To **process events**, create a consumer function:

```python
def consume_events():
last_id = '0'
while True:
events = redis_client.xread({STREAM_NAME: last_id}, block=0, count=5)
for stream, messages in events:
for message in messages:
last_id, event_data = message
print(f"Processing event: {event_data}")
```

This function will continuously **consume and process real-time events**.

### Running the Consumer as a Background Task

To run the consumer **in the background**, start it in a separate thread:

```python
import threading

threading.Thread(target=consume_events, daemon=True).start()
```

This will keep listening for **new real-time events** without blocking the Flask app.

## Scaling Redis Streams with Consumer Groups

For **high-throughput real-time processing**, use **consumer groups**:

### Creating a Consumer Group

```python
GROUP_NAME = "event_consumers"

try:
redis_client.xgroup_create(STREAM_NAME, GROUP_NAME, id='0', mkstream=True)
except redis.exceptions.ResponseError:
pass
```

### Processing Messages Using a Consumer Group

```python
def consume_from_group(consumer_name):
while True:
events = redis_client.xreadgroup(GROUP_NAME, consumer_name, {STREAM_NAME: ">"}, count=5, block=5000)
for stream, messages in events:
for message in messages:
msg_id, event_data = message
print(f"[{consumer_name}] Processing: {event_data}")
redis_client.xack(STREAM_NAME, GROUP_NAME, msg_id)
```

### Running Multiple Consumers

```sh
python consumer.py --consumer-name=worker1 &
python consumer.py --consumer-name=worker2 &
```

This setup enables **distributed event processing** using multiple consumers.

## Conclusion

**Flask and Redis Streams** provide a powerful solution for **real-time event-driven applications**. By using **streaming data ingestion, consumer groups, and background processing**, you can build **scalable and efficient real-time systems**.

Start integrating **Redis Streams with Flask** today and unlock the power of **real-time data processing**! 🚀  
