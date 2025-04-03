---
layout: post
title: "Flask and MongoDB: Working with NoSQL Databases"
subtitle: "Learn how to integrate MongoDB with Flask for scalable NoSQL database management"
categories: Flask
tags: ["Flask", "MongoDB", "NoSQL", "Database", "PyMongo", "API", "CRUD"]
excerpt: "Explore how to connect Flask with MongoDB, perform CRUD operations, and build a scalable NoSQL-based web application with PyMongo."
---



MongoDB is a **NoSQL database** that offers **scalability, flexibility, and high performance** for modern applications. Unlike relational databases, MongoDB stores data in **JSON-like documents**, making it a great choice for **dynamic applications**.

In this tutorial, we will explore:

✅ **Setting up MongoDB with Flask**  
✅ **Using PyMongo to interact with MongoDB**  
✅ **Performing CRUD operations (Create, Read, Update, Delete)**  
✅ **Building a simple API with Flask and MongoDB**

---

## Step 1: Install MongoDB and PyMongo

### Install MongoDB

MongoDB must be installed and running. For local development, you can:

🔹 Install MongoDB from [MongoDB’s official site](https://www.mongodb.com/try/download/community)  
🔹 Use **Docker**:

```sh
docker run -d -p 27017:27017 --name mongodb mongo:latest
```

### Install PyMongo

PyMongo is the official Python driver for MongoDB:

```sh
pip install flask pymongo
```

---

## Step 2: Configure Flask and MongoDB

Create a **Flask application** and configure MongoDB connection:

```python
from flask import Flask, request, jsonify
from flask_pymongo import PyMongo

app = Flask(__name__)
app.config["MONGO_URI"] = "mongodb://localhost:27017/mydatabase"
mongo = PyMongo(app)
```

🔹 **`MONGO_URI`** specifies the database connection  
🔹 **`PyMongo(app)`** initializes MongoDB with Flask

---

## Step 3: Define a Data Model

MongoDB does not enforce schemas, but we can define a **structured document format** in our application.

For example, a **user document** might look like:

```json
{
"name": "John Doe",
"email": "john@example.com",
"age": 30
}
```

---

## Step 4: Implement CRUD Operations

### **Create (Insert a New User)**

```python
@app.route("/users", methods=["POST"])
def create_user():
data = request.json
user_id = mongo.db.users.insert_one(data).inserted_id
return jsonify({"message": "User added", "id": str(user_id)})
```

📌 **`insert_one(data)`** adds a new document to the collection

---

### **Read (Get All Users)**

```python
@app.route("/users", methods=["GET"])
def get_users():
users = list(mongo.db.users.find({}, {"_id": 0}))
return jsonify(users)
```

📌 **`find({}, {"_id": 0})`** fetches all users while **excluding** `_id`

---

### **Update (Modify an Existing User)**

```python
@app.route("/users/<email>", methods=["PUT"])
def update_user(email):
data = request.json
result = mongo.db.users.update_one({"email": email}, {"$set": data})
return jsonify({"message": "User updated", "matched": result.matched_count})
```

📌 **`update_one()`** modifies documents based on **email**

---

### **Delete (Remove a User)**

```python
@app.route("/users/<email>", methods=["DELETE"])
def delete_user(email):
result = mongo.db.users.delete_one({"email": email})
return jsonify({"message": "User deleted", "deleted": result.deleted_count})
```

📌 **`delete_one()`** removes a document matching the query

---

## Step 5: Indexing and Performance Optimization

For faster queries, create **indexes**:

```python
mongo.db.users.create_index("email", unique=True)
```

🔹 **Indexes improve search performance**  
🔹 **`unique=True`** prevents duplicate emails

---

## Step 6: Running the Flask App

Start the Flask server:

```sh
flask run --host=0.0.0.0 --port=5000
```

Now, test the API using **Postman** or **cURL**.

---

## Conclusion

🔹 **Flask + MongoDB = Scalable NoSQL Web Apps**  
🔹 **PyMongo makes database interactions easy**  
🔹 **CRUD operations help manage data efficiently**

By following this guide, you can integrate MongoDB with Flask and build powerful NoSQL applications! 🚀

---
