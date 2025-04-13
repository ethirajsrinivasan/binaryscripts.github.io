---
layout: post
title: "Implementing JWT Authentication and Authorization in Flask"
subtitle: "Secure your Flask APIs with JWT authentication and role-based authorization"
categories: Flask
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: ["Flask", "JWT", "Authentication", "Authorization", "Security", "REST", "API"]
excerpt: "Learn how to implement JWT authentication and authorization in Flask to secure your APIs. Understand token-based authentication, role-based access control, and best practices for securing API endpoints."
---
In modern web applications, securing APIs is **critical** to prevent unauthorized access. One of the most effective ways to manage authentication and authorization is using **JWT (JSON Web Tokens)**.

JWT provides a **stateless** way to authenticate users, eliminating the need for session storage while ensuring secure access control. In this blog, we'll cover **how to implement JWT authentication and authorization in Flask**, including **token generation, verification, and role-based access control**.

## Setting Up Flask and Required Dependencies

Before diving into JWT implementation, install the necessary dependencies:

```sh
pip install flask flask-jwt-extended
```

Then, initialize a Flask app and configure JWT settings:

```python
from flask import Flask, jsonify, request
from flask_jwt_extended import (
JWTManager, create_access_token, jwt_required, get_jwt_identity
)

app = Flask(__name__)
app.config['JWT_SECRET_KEY'] = 'your_secret_key'  # Change this to a strong secret key

jwt = JWTManager(app)
```

## Implementing User Authentication

Create a simple login endpoint that validates user credentials and returns a JWT token:

```python
users = {
"admin": {"password": "adminpass", "role": "admin"},
"user": {"password": "userpass", "role": "user"}
}

@app.route('/login', methods=['POST'])
def login():
auth_data = request.json
username = auth_data.get("username")
password = auth_data.get("password")

    if username in users and users[username]["password"] == password:
        access_token = create_access_token(identity={"username": username, "role": users[username]["role"]})
        return jsonify(access_token=access_token)

    return jsonify({"message": "Invalid credentials"}), 401
```

This function validates **username and password**. If correct, it returns a JWT **access token**.

## Protecting Routes with JWT

To protect an API route, use the **@jwt_required()** decorator:

```python
@app.route('/protected', methods=['GET'])
@jwt_required()
def protected():
current_user = get_jwt_identity()
return jsonify(message=f"Hello, {current_user['username']}!"), 200
```

If the user provides a valid **JWT token**, they can access this route. Otherwise, they receive a **401 Unauthorized** error.

## Implementing Role-Based Authorization

For advanced security, restrict API access based on user roles:

```python
from functools import wraps

def role_required(required_role):
def decorator(fn):
@wraps(fn)
@jwt_required()
def wrapper(*args, **kwargs):
current_user = get_jwt_identity()
if current_user["role"] != required_role:
return jsonify({"message": "Access denied"}), 403
return fn(*args, **kwargs)
return wrapper
return decorator

@app.route('/admin-only', methods=['GET'])
@role_required("admin")
def admin_dashboard():
return jsonify(message="Welcome, Admin!"), 200
```

This **role-based access control (RBAC)** ensures only **admins** can access the `/admin-only` endpoint.

## Token Expiry and Refresh

For security, JWT tokens should expire after a set period. To enable token refresh, modify the configuration:

```python
from flask_jwt_extended import create_refresh_token, jwt_refresh_token_required

app.config['JWT_ACCESS_TOKEN_EXPIRES'] = 900  # 15 minutes
app.config['JWT_REFRESH_TOKEN_EXPIRES'] = 86400  # 24 hours

@app.route('/refresh', methods=['POST'])
@jwt_refresh_token_required
def refresh():
current_user = get_jwt_identity()
new_token = create_access_token(identity=current_user)
return jsonify(access_token=new_token)
```

This allows users to request a **new access token** using a refresh token when their current token expires.

## Conclusion

JWT authentication and authorization provide a **secure, scalable, and stateless** way to protect Flask APIs. By implementing token validation, **role-based access control (RBAC)**, and **refresh tokens**, you can enhance your API's security and user management.

Start integrating JWT authentication into your Flask applications today! 🚀  
