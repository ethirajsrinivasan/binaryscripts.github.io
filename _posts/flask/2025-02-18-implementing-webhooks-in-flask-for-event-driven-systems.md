---
layout: post
title: "Implementing Webhooks in Flask for Event-Driven Systems"
subtitle: "Learn how to build and handle webhooks in Flask for real-time event-driven applications"
categories: Flask
tags: ["Flask", "Webhooks", "Event-Driven", "API", "Automation", "Python"]
excerpt: "Discover how to implement webhooks in Flask to handle real-time event-driven communication. Learn about request validation, security, and best practices."
---
**Webhooks** enable real-time communication between systems by **sending HTTP callbacks** when events occur. They play a crucial role in **event-driven architectures**, allowing applications to react instantly to changes.

In this guide, you will learn:

✅ **How webhooks work**  
✅ **Setting up webhook endpoints in Flask**  
✅ **Validating and securing webhook requests**  
✅ **Handling retries and failures**

## What Are Webhooks?

A **webhook** is an **HTTP callback** triggered by an event in one system, notifying another system via a **POST request**.

### 🔹 Webhooks vs. APIs

| Feature       | Webhooks | Traditional APIs |
|--------------|---------|-----------------|
| Communication | Event-driven | Request-based |
| Efficiency    | Real-time updates | Polling required |
| Use Case     | Notifications, automation | Data retrieval, CRUD operations |

## Setting Up a Webhook in Flask

### Installing Dependencies

Ensure you have Flask installed:

```bash
pip install flask
```

### Creating the Webhook Endpoint

Create a new file `app.py` and set up a basic Flask webhook listener:

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/webhook", methods=["POST"])
def webhook():
data = request.json
if not data:
return jsonify({"error": "Invalid payload"}), 400

    # Log and process the webhook event
    print("Received Webhook:", data)
    
    return jsonify({"message": "Webhook received"}), 200

if __name__ == "__main__":
app.run(port=5000, debug=True)
```

Start the Flask app:

```bash
python app.py
```

Now, the **webhook URL** is `http://localhost:5000/webhook`.

### Testing with cURL

Simulate an event using cURL:

```bash
curl -X POST http://localhost:5000/webhook -H "Content-Type: application/json" -d '{"event": "user_signed_up", "user": "john_doe"}'
```

## Validating Webhook Requests

### Using Secret Tokens

To **prevent unauthorized webhook calls**, verify a **secret token**:

```python
import hmac
import hashlib

SECRET_TOKEN = "your_secret_key"

def verify_signature(payload, signature):
expected_signature = hmac.new(SECRET_TOKEN.encode(), payload, hashlib.sha256).hexdigest()
return hmac.compare_digest(expected_signature, signature)

@app.route("/webhook", methods=["POST"])
def webhook():
signature = request.headers.get("X-Signature")
payload = request.data

    if not verify_signature(payload, signature):
        return jsonify({"error": "Invalid signature"}), 403

    data = request.json
    print("Verified Webhook:", data)
    
    return jsonify({"message": "Webhook received"}), 200
```

### Handling Replay Attacks

Use timestamps to prevent replay attacks:

```python
import time

@app.route("/webhook", methods=["POST"])
def webhook():
timestamp = request.headers.get("X-Timestamp")

    if abs(time.time() - int(timestamp)) > 300:
        return jsonify({"error": "Request expired"}), 400

    # Proceed with webhook processing
```

## Retrying Failed Webhooks

If the receiver fails to process a webhook, **automatic retries** ensure delivery:

- Return **HTTP 200** for successful processing
- Return **HTTP 500** to trigger retries

Modify `app.py`:

```python
@app.route("/webhook", methods=["POST"])
def webhook():
data = request.json

    # Simulate processing failure
    if data.get("event") == "fail_test":
        return jsonify({"error": "Simulated failure"}), 500

    return jsonify({"message": "Webhook received"}), 200
```

Test failure handling:

```bash
curl -X POST http://localhost:5000/webhook -H "Content-Type: application/json" -d '{"event": "fail_test"}'
```

## Logging Webhook Events

Log webhook activity using Python’s logging module:

```python
import logging

logging.basicConfig(filename="webhook.log", level=logging.INFO)

@app.route("/webhook", methods=["POST"])
def webhook():
data = request.json
logging.info(f"Received Webhook: {data}")
return jsonify({"message": "Logged webhook"}), 200
```

## Deploying the Webhook Listener

### Running with Gunicorn

Use **Gunicorn** for production:

```bash
pip install gunicorn
gunicorn -w 4 app:app
```

### Exposing the Webhook with ngrok

Expose your local webhook to the internet using **ngrok**:

```bash
ngrok http 5000
```

Copy the **ngrok URL** and configure it in the webhook provider.

## Conclusion

With **Flask webhooks**, you can build real-time event-driven applications. This guide covered:

✅ **How webhooks work**  
✅ **Setting up Flask webhook endpoints**  
✅ **Securing webhooks with tokens**  
✅ **Handling failures and retries**

Start integrating webhooks into your **Flask applications today! 🚀**  
