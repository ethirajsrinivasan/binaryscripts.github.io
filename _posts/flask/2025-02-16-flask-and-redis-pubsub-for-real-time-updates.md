---
layout: post
title: "Flask and Redis Pub/Sub for Real-Time Updates"
subtitle: "Using Flask and Redis Pub/Sub for real-time event-driven communication"
categories: Flask
tags: ["Flask", "Redis", "Pub/Sub", "WebSockets", "Real-Time", "Streaming"]
excerpt: "Learn how to integrate Flask with Redis Pub/Sub for real-time messaging and event-driven communication, enabling instant updates and scalable architectures."
---

## Introduction

Real-time applications are essential for **live notifications, chat applications, and event-driven architectures**. **Flask**, when combined with **Redis Pub/Sub**, enables instant **message broadcasting** and **event handling** without the complexity of traditional WebSocket setups.

### What We’ll Cover
✅ Setting up Redis Pub/Sub with Flask  
✅ Publishing and subscribing to messages  
✅ Handling real-time updates in a Flask application

---

## Why Use Redis Pub/Sub?

Redis **Publish/Subscribe (Pub/Sub)** is a lightweight **messaging system** that allows applications to send and receive messages in real time.

🔹 **Decoupled Communication** - Publishers and subscribers do not need to know each other.  
🔹 **Low Latency** - Messages are delivered instantly.  
🔹 **Scalability** - Multiple subscribers can receive the same event.

---

## Step 1: Installing Dependencies

Ensure you have Flask and Redis installed.

```sh
pip install flask redis
```

You also need **Redis** running. Start Redis using:

```sh
redis-server
```

---

## Step 2: Setting Up Flask

### Creating a Basic Flask App

```python
from flask import Flask, request, jsonify
import redis

app = Flask(__name__)
redis_client = redis.StrictRedis(host="localhost", port=6379, decode_responses=True)

@app.route("/")
def home():
return "Flask & Redis Pub/Sub Example"

if __name__ == "__main__":
app.run(debug=True)
```

---

## Step 3: Implementing Redis Pub/Sub

### 1. **Publishing Messages**

A publisher **sends messages** to a Redis channel.

```python
@app.route("/publish", methods=["POST"])
def publish_message():
data = request.json
message = data.get("message", "")

    redis_client.publish("updates", message)
    return jsonify({"status": "Message published"})
```

### 2. **Subscribing to Messages**

A subscriber **listens** for messages from a Redis channel.

```python
def message_listener():
pubsub = redis_client.pubsub()
pubsub.subscribe("updates")

    for message in pubsub.listen():
        if message["type"] == "message":
            print(f"Received: {message['data']}")

# Run the listener in a background thread
import threading
threading.Thread(target=message_listener, daemon=True).start()
```

---

## Step 4: Testing Redis Pub/Sub

### Publishing a Message

Use **cURL** or Postman to send a message:

```sh
curl -X POST http://127.0.0.1:5000/publish -H "Content-Type: application/json" -d '{"message": "Hello, Redis!"}'
```

Check the console logs. The subscriber should **receive the message instantly**!

---

## Step 5: Integrating with WebSockets for Real-Time Updates

While Redis handles messaging, we can use **Flask-SocketIO** to push updates to the frontend.

### Install Flask-SocketIO

```sh
pip install flask-socketio
```

### Modify Flask to Use WebSockets

```python
from flask_socketio import SocketIO

socketio = SocketIO(app, cors_allowed_origins="*")

def message_listener():
pubsub = redis_client.pubsub()
pubsub.subscribe("updates")

    for message in pubsub.listen():
        if message["type"] == "message":
            socketio.emit("update", {"data": message["data"]})

# Run the WebSocket server
if __name__ == "__main__":
threading.Thread(target=message_listener, daemon=True).start()
socketio.run(app, debug=True)
```

---

## Step 6: Frontend Example (JavaScript)

The frontend can receive real-time updates using **Socket.IO**.

```html
<script src="https://cdn.socket.io/4.0.0/socket.io.js"></script>
<script>
    var socket = io("http://127.0.0.1:5000");

    socket.on("update", function (data) {
        console.log("New Update: ", data);
    });
</script>
```

---

## Step 7: Scaling with Redis Cluster

For production, use **Redis Cluster** to improve performance:

- **Run multiple Redis instances**
- **Use a message broker like RabbitMQ**
- **Deploy on AWS ElastiCache**

---

## Conclusion

By combining **Flask + Redis Pub/Sub**, we create a **lightweight real-time architecture** suitable for:

🚀 **Live notifications**  
💬 **Chat applications**  
📡 **IoT event streaming**

Start building **real-time Flask apps today!** 🚀  
