---
layout: post
title: "Integrating Flask with Celery for Background Processing"
subtitle: "Efficiently handle long-running tasks without blocking your Flask application"
categories: Flask
tags: ["Flask", "Celery", "Background Tasks", "Asynchronous Processing", "Python"]
excerpt: "Learn how to integrate Celery with Flask for executing long-running tasks asynchronously, improving scalability and performance."
---
Flask is a lightweight framework for building web applications, but it is **synchronous by default**, meaning requests are processed one at a time. This can be a limitation when handling **long-running tasks** like:

- Sending emails
- Generating reports
- Processing large datasets
- Running scheduled jobs

To handle such tasks efficiently, we can **integrate Celery** with Flask. Celery allows us to process tasks **asynchronously** in the background while keeping the Flask app responsive.

---

## Step 1: Install Dependencies

To get started, install Flask and Celery:

```sh
pip install flask celery redis
```

Celery requires a **message broker** to handle task queues. We’ll use **Redis** (ensure it's installed and running):

```sh
redis-server
```

---

## Step 2: Setting Up Flask and Celery

Create a **Flask app (`app.py`)** and initialize Celery:

```python
from flask import Flask, request, jsonify
from celery import Celery

app = Flask(__name__)

# Celery configuration
app.config["CELERY_BROKER_URL"] = "redis://localhost:6379/0"
app.config["CELERY_RESULT_BACKEND"] = "redis://localhost:6379/0"

celery = Celery(app.name, broker=app.config["CELERY_BROKER_URL"])
celery.conf.update(app.config)
```

---

## Step 3: Define an Asynchronous Task

Create a Celery task to run in the background:

```python
@celery.task
def long_task(duration):
import time
time.sleep(duration)  # Simulating a long-running process
return f"Task completed in {duration} seconds"
```

---

## Step 4: Create an API Endpoint to Trigger the Task

Define an endpoint to **start the background task**:

```python
@app.route("/start-task", methods=["POST"])
def start_task():
duration = request.json.get("duration", 5)
task = long_task.apply_async(args=[duration])
return jsonify({"task_id": task.id, "status": "Task started"})
```

---

## Step 5: Monitor Task Status

Since tasks are executed asynchronously, we need an endpoint to check task progress:

```python
@app.route("/task-status/<task_id>", methods=["GET"])
def task_status(task_id):
task = long_task.AsyncResult(task_id)
return jsonify({"task_id": task.id, "status": task.status, "result": task.result})
```

---

## Step 6: Run Flask and Celery

Start the Flask application:

```sh
python app.py
```

In a separate terminal, start the Celery worker:

```sh
celery -A app.celery worker --loglevel=info
```

This will listen for background tasks and process them asynchronously.

---

## Step 7: Test the Setup

1. **Start a background task:**

```sh
curl -X POST http://127.0.0.1:5000/start-task -H "Content-Type: application/json" -d '{"duration": 10}'
```

2. **Get the task ID from the response** and check its status:

```sh
curl http://127.0.0.1:5000/task-status/<task_id>
```

---

## Step 8: Enhancing Celery for Production

### Periodic Tasks with Celery Beat

To run scheduled tasks, install Celery Beat:

```sh
pip install celery[redis] celery-beat
```

Create a **periodic task**:

```python
from celery.schedules import crontab

celery.conf.beat_schedule = {
"run-every-minute": {
"task": "app.long_task",
"schedule": crontab(minute="*/1"),  # Runs every minute
"args": [5]
}
}
```

Start Celery Beat:

```sh
celery -A app.celery beat --loglevel=info
```

---

## Step 9: Deploying Flask + Celery

For **production**, consider:

✅ **Using Docker**: Containerize Flask and Celery with Redis  
✅ **Using Supervisor**: Automatically restart Celery workers  
✅ **Using a Distributed Task Queue**: Scale with multiple workers

Example **Docker Compose setup** for Flask, Celery, and Redis:

```yaml
version: '3'

services:
redis:
image: redis:latest
container_name: redis
ports:
- "6379:6379"

flask-app:
build: .
container_name: flask-app
ports:
- "5000:5000"
depends_on:
- redis

celery-worker:
build: .
container_name: celery-worker
command: celery -A app.celery worker --loglevel=info
depends_on:
- redis
```

Run everything with:

```sh
docker-compose up --build
```

---

## Conclusion

By integrating **Celery with Flask**, we can execute long-running tasks **asynchronously**, improving app performance and scalability. This is crucial for modern web applications that require background processing, such as:

🔹 Sending emails  
🔹 Data processing  
🔹 Image processing  
🔹 Scheduled jobs

💡 **Now you can build scalable Flask applications with Celery! 🚀**  
