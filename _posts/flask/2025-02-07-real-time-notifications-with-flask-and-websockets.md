---
layout: post
title: "Real-Time Notifications with Flask and WebSockets"
subtitle: "Learn how to build a real-time notification system using Flask, WebSockets, and Flask-SocketIO"
categories: Flask
tags: ["Flask", "WebSockets", "Real-Time", "Notifications", "Flask-SocketIO", "Python"]
excerpt: "Discover how to implement a real-time notification system in Flask using WebSockets and Flask-SocketIO for instant data updates and seamless user experience."
---



Traditional HTTP-based web applications rely on **request-response cycles**, meaning users only receive updates when they refresh the page. However, for applications that require **real-time notifications**, such as:

- Chat applications
- Live stock price updates
- Collaborative tools
- IoT dashboards

Using **WebSockets** with Flask allows **bi-directional, real-time communication** between clients and the server. In this guide, we’ll integrate **Flask-SocketIO** to send notifications instantly to connected users.

---

## Step 1: Install Dependencies

To get started, install Flask and Flask-SocketIO:

```sh
pip install flask flask-socketio eventlet
```

We use **eventlet** as the asynchronous server, which is required for handling multiple real-time connections.

---

## Step 2: Setting Up Flask with WebSockets

Create a **Flask application (`app.py`)** and configure WebSockets:

```python
from flask import Flask, render_template
from flask_socketio import SocketIO, emit

app = Flask(__name__)
app.config["SECRET_KEY"] = "your_secret_key"
socketio = SocketIO(app, cors_allowed_origins="*")

@app.route("/")
def index():
return render_template("index.html")

@socketio.on("connect")
def handle_connect():
print("Client connected")

@socketio.on("disconnect")
def handle_disconnect():
print("Client disconnected")

if __name__ == "__main__":
socketio.run(app, debug=True, host="0.0.0.0", port=5000)
```

- **`socketio.on("connect")`**: Detects when a client connects.
- **`socketio.on("disconnect")`**: Detects when a client disconnects.

---

## Step 3: Creating the Frontend

In the `templates` folder, create an **`index.html`** file to handle WebSocket connections:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Real-Time Notifications</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.3.2/socket.io.js"></script>
</head>
<body>
    <h1>Real-Time Notifications with Flask-SocketIO</h1>
    <button onclick="sendNotification()">Send Notification</button>
    <p id="notification"></p>

    <script>
        var socket = io.connect("http://" + document.domain + ":" + location.port);

        socket.on("new_notification", function(data) {
            document.getElementById("notification").innerText = data.message;
        });

        function sendNotification() {
            socket.emit("send_notification", {message: "New Notification Received!"});
        }
    </script>
</body>
</html>
```  

---

## Step 4: Broadcasting Notifications

Modify `app.py` to handle real-time notifications:

```python
@socketio.on("send_notification")
def handle_notification(data):
emit("new_notification", {"message": data["message"]}, broadcast=True)
```

- **`emit("new_notification", ..., broadcast=True)`**: Sends the message to **all connected clients** in real-time.

---

## Step 5: Running the Flask WebSocket Server

Start the Flask app:

```sh
python app.py
```

Now, open multiple browser tabs at **`http://127.0.0.1:5000/`**, and press the **Send Notification** button. The message will appear in **all tabs simultaneously**, demonstrating real-time updates.

---

## Step 6: Using WebSockets with Celery for Background Tasks

For applications that require **asynchronous notifications**, such as sending emails or processing user activity, integrate **Celery** with Flask-SocketIO.

### Install Celery

```sh
pip install celery redis
```

### Configure Celery in `app.py`

```python
from celery import Celery

app.config["CELERY_BROKER_URL"] = "redis://localhost:6379/0"
app.config["CELERY_RESULT_BACKEND"] = "redis://localhost:6379/0"

celery = Celery(app.name, broker=app.config["CELERY_BROKER_URL"])
celery.conf.update(app.config)

@celery.task
def send_async_notification(message):
socketio.emit("new_notification", {"message": message}, broadcast=True)
```

Now, trigger notifications **asynchronously**:

```python
@app.route("/trigger-task")
def trigger_task():
send_async_notification.apply_async(args=["Background Task Notification"])
return "Task triggered!"
```

Start the **Celery worker**:

```sh
celery -A app.celery worker --loglevel=info
```

Now, when you visit **`http://127.0.0.1:5000/trigger-task`**, a notification will be sent **without blocking the main thread**.

---

## Step 7: Deploying Flask WebSockets

For production deployment, use:

✅ **Gunicorn with Eventlet**: Handles multiple WebSocket connections.  
✅ **Nginx as a Reverse Proxy**: Routes WebSocket traffic efficiently.  
✅ **Docker for Scalability**: Containerizes the application.

### Running with Gunicorn

Install **Gunicorn**:

```sh
pip install gunicorn
```

Run Flask with Gunicorn + Eventlet:

```sh
gunicorn -k eventlet -w 1 app:app --bind 0.0.0.0:5000
```

This ensures WebSocket **connections remain persistent**.

---

## Conclusion

By integrating **Flask-SocketIO**, we can build **real-time notification systems** that update users instantly. This approach is useful for **chats, alerts, and live data streaming**.

### Key Takeaways:

✅ **WebSockets enable bi-directional, real-time communication**  
✅ **Flask-SocketIO simplifies WebSocket handling in Flask**  
✅ **Celery can offload background notifications asynchronously**  
✅ **Gunicorn + Eventlet ensure WebSocket scalability in production**

🚀 **Now you can add real-time notifications to your Flask app!**  
