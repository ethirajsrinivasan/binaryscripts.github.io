---
layout: post
title: "Using Flask with WebSockets for Real-Time Applications"
subtitle: "Learn how to integrate WebSockets with Flask to build real-time applications"
categories: Flask
tags: ["Flask", "WebSockets", "Real-Time", "Python", "SocketIO", "Microservices"]
excerpt: "Discover how to use Flask with WebSockets to build real-time applications. Learn about Flask-SocketIO, event handling, broadcasting, and performance optimizations."
---
Traditional HTTP-based applications rely on request-response cycles, making **real-time interactions** challenging. WebSockets solve this problem by providing **full-duplex communication** between clients and servers.

In this guide, we'll explore how to **integrate WebSockets with Flask** using `Flask-SocketIO` to build **real-time applications** like chat apps, notifications, and live dashboards.

## Setting Up Flask with WebSockets

#### Installing Dependencies

First, install the required packages:

```sh
pip install flask flask-socketio eventlet
```

- `Flask-SocketIO`: Enables WebSockets in Flask.
- `eventlet`: A high-performance networking library for real-time apps.

#### Creating a Basic WebSocket Server

```python
from flask import Flask, render_template
from flask_socketio import SocketIO

app = Flask(__name__)
socketio = SocketIO(app, cors_allowed_origins="*")

@app.route('/')
def index():
return "WebSocket Server is Running!"

@socketio.on('message')
def handle_message(msg):
print(f"Received message: {msg}")
socketio.send(f"Echo: {msg}")

if __name__ == '__main__':
socketio.run(app, debug=True, host='0.0.0.0', port=5000)
```

- This **creates a WebSocket server** that listens for messages and echoes them back.
- The `@socketio.on('message')` decorator handles incoming messages.

## Building a Real-Time Chat Application

Let's extend this to a **real-time chat app** where multiple users can communicate.

#### Frontend HTML

Save this as `templates/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Flask WebSocket Chat</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.5.0/socket.io.js"></script>
</head>
<body>
    <h2>Flask WebSocket Chat</h2>
    <input id="msg" type="text" placeholder="Type a message..." />
    <button onclick="sendMessage()">Send</button>
    <ul id="messages"></ul>

    <script>
        var socket = io.connect('http://' + document.domain + ':' + location.port);

        socket.on('message', function(msg) {
            var li = document.createElement("li");
            li.appendChild(document.createTextNode(msg));
            document.getElementById("messages").appendChild(li);
        });

        function sendMessage() {
            var msg = document.getElementById("msg").value;
            socket.send(msg);
        }
    </script>
</body>
</html>
```  

- This page **connects to the WebSocket server** and sends messages.
- Messages are appended to the chat window in real-time.

## Broadcasting Messages to All Clients

Modify the server to **broadcast messages** to all connected clients:

```python
@socketio.on('message')
def handle_message(msg):
print(f"Received message: {msg}")
socketio.emit('message', msg, broadcast=True)
```

Now, **all connected clients** receive new messages instantly.

## Handling Custom Events

You can define **custom WebSocket events**:

```python
@socketio.on('join')
def handle_join(data):
username = data['username']
socketio.emit('message', f"{username} has joined the chat", broadcast=True)
```

Client-side:

```js
socket.emit('join', {username: 'Alice'});
```

## Scaling WebSocket Applications

For large-scale apps, use **message queues like Redis**:

```sh
pip install redis
```

Modify the server to use Redis as a message queue:

```python
socketio = SocketIO(app, cors_allowed_origins="*", message_queue="redis://")
```

This allows **multiple instances of Flask** to communicate efficiently.

## Conclusion

Integrating WebSockets with Flask enables **real-time communication** for applications like:

- Chat applications
- Live dashboards
- Multiplayer games
- Instant notifications

By using **Flask-SocketIO, event handling, and Redis**, you can build **scalable and efficient** real-time applications. 🚀  
