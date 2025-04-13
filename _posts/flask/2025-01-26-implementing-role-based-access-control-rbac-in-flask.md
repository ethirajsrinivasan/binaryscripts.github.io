---
layout: post
title: "Implementing Role-Based Access Control (RBAC) in Flask"
subtitle: "Secure your Flask applications with Role-Based Access Control (RBAC) for fine-grained user permissions"
categories: Flask
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: ["Flask", "RBAC", "Authentication", "Authorization", "Security", "JWT", "SQLAlchemy"]
excerpt: "Learn how to implement Role-Based Access Control (RBAC) in Flask to enhance security by defining user roles, managing permissions, and integrating JWT authentication for protected routes."
---
**Role-Based Access Control (RBAC)** is a security mechanism that restricts access based on predefined **user roles**. In Flask, implementing RBAC ensures that users can only perform actions authorized by their roles, **enhancing security and maintainability**.

In this guide, we’ll build an **RBAC system** using **Flask, Flask-Login, SQLAlchemy, and JWT authentication**.

## Setting Up Flask and Dependencies

### Install Required Packages

```bash
pip install flask flask-sqlalchemy flask-login flask-jwt-extended
```

### Initialize Flask App

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager
from flask_jwt_extended import JWTManager

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///rbac.db'
app.config['JWT_SECRET_KEY'] = 'supersecretkey'

db = SQLAlchemy(app)
login_manager = LoginManager(app)
jwt = JWTManager(app)
```

## Creating User and Role Models

Define models for **User** and **Role** using SQLAlchemy.

```python
class Role(db.Model):
id = db.Column(db.Integer, primary_key=True)
name = db.Column(db.String(50), unique=True, nullable=False)

class User(db.Model):
id = db.Column(db.Integer, primary_key=True)
username = db.Column(db.String(100), unique=True, nullable=False)
password = db.Column(db.String(100), nullable=False)
role_id = db.Column(db.Integer, db.ForeignKey('role.id'), nullable=False)

    role = db.relationship('Role', backref=db.backref('users', lazy=True))
```

✅ **Each user is assigned a role**  
✅ **Roles define access permissions**

## Seeding Roles into the Database

Before enforcing RBAC, seed roles into the database.

```python
def seed_roles():
admin = Role(name="admin")
user = Role(name="user")
db.session.add(admin)
db.session.add(user)
db.session.commit()

if __name__ == "__main__":
db.create_all()
seed_roles()
```

## Implementing Authentication

### Hashing Passwords

Use **Werkzeug’s password hashing** for security.

```bash
pip install werkzeug
```

```python
from werkzeug.security import generate_password_hash, check_password_hash

def create_user(username, password, role):
hashed_password = generate_password_hash(password)
user = User(username=username, password=hashed_password, role=role)
db.session.add(user)
db.session.commit()
```

### User Login and JWT Token Generation

```python
from flask import request, jsonify
from flask_jwt_extended import create_access_token

@app.route('/login', methods=['POST'])
def login():
data = request.json
user = User.query.filter_by(username=data['username']).first()

    if user and check_password_hash(user.password, data['password']):
        access_token = create_access_token(identity={"id": user.id, "role": user.role.name})
        return jsonify(access_token=access_token), 200

    return jsonify({"msg": "Invalid credentials"}), 401
```

✅ **Generates JWT token upon login**  
✅ **Token contains user ID and role**

## Enforcing Role-Based Access

### Custom RBAC Decorator

```python
from functools import wraps
from flask_jwt_extended import get_jwt_identity, jwt_required

def role_required(role):
def decorator(func):
@wraps(func)
@jwt_required()
def wrapper(*args, **kwargs):
identity = get_jwt_identity()
if identity['role'] != role:
return jsonify({"msg": "Access denied"}), 403
return func(*args, **kwargs)
return wrapper
return decorator
```

### Protecting Routes with RBAC

```python
@app.route('/admin', methods=['GET'])
@role_required('admin')
def admin_panel():
return jsonify({"msg": "Welcome Admin!"})

@app.route('/user', methods=['GET'])
@role_required('user')
def user_dashboard():
return jsonify({"msg": "Welcome User!"})
```

✅ **Restricts access based on role**  
✅ **JWT protects endpoints**

## Testing RBAC Implementation

### Create an Admin and User

```python
admin_role = Role.query.filter_by(name="admin").first()
user_role = Role.query.filter_by(name="user").first()

create_user("admin_user", "securepass", admin_role)
create_user("regular_user", "securepass", user_role)
```

### Obtain JWT Tokens

Login via API:

```bash
curl -X POST http://127.0.0.1:5000/login -H "Content-Type: application/json" -d '{"username": "admin_user", "password": "securepass"}'
```

Response:

```json
{
"access_token": "your_generated_jwt_token"
}
```

### Access Admin Route

```bash
curl -X GET http://127.0.0.1:5000/admin -H "Authorization: Bearer your_generated_jwt_token"
```

✅ **Returns 200 for Admin**  
✅ **Returns 403 for unauthorized users**

## Conclusion

Implementing **Role-Based Access Control (RBAC)** in Flask enhances **security and access control**. This guide covered:

✔ **Setting up Flask and dependencies**  
✔ **Defining User and Role models**  
✔ **Implementing authentication with JWT**  
✔ **Creating an RBAC decorator for role-based restrictions**  
✔ **Testing secured routes with user roles**

By integrating RBAC, your Flask application becomes **more secure, scalable, and maintainable**. 🚀  
