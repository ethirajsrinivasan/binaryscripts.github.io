---
layout: post
title: "Creating Multi-Tenancy in Flask Applications"
subtitle: "Implementing Multi-Tenancy with Database Isolation and Schema Separation in Flask"
categories: Flask
tags: ["Flask", "Multi-Tenancy", "SQLAlchemy", "Database", "Web Development"]
excerpt: "Learn how to design and implement a multi-tenant architecture in Flask using database isolation and schema-based approaches."
---
Multi-tenancy is a **software architecture pattern** that allows a **single application instance** to serve multiple **customers (tenants)** while keeping their data isolated. In this guide, we'll explore different **multi-tenancy strategies** in Flask, including:

- **Database-per-tenant**
- **Schema-based multi-tenancy**
- **Row-level multi-tenancy**

---

## **Why Multi-Tenancy?**

Multi-tenancy is crucial for **SaaS (Software as a Service)** applications, as it allows:

✅ **Cost Efficiency** – One codebase serves multiple tenants.  
✅ **Data Isolation** – Tenant data is segregated for security.  
✅ **Scalability** – Easily add new tenants without changing the core system.

---

## **Setting Up Flask with SQLAlchemy**

### **Install Dependencies**

Ensure you have Flask and SQLAlchemy installed:

```sh
pip install flask flask_sqlalchemy flask-migrate psycopg2
```

### **Flask App Configuration**

```python
from flask import Flask, g, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# Default Database URI (used if tenant-specific DB is not set)
app.config["SQLALCHEMY_DATABASE_URI"] = "postgresql://user:password@localhost/default_db"
db = SQLAlchemy(app)

class Tenant(db.Model):
id = db.Column(db.Integer, primary_key=True)
name = db.Column(db.String(50), unique=True, nullable=False)
database_url = db.Column(db.String(200), nullable=False)
```

### **Determining Tenant from Request**

Each request must be linked to a **specific tenant**. We'll extract the tenant identifier from the request **subdomain or headers**:

```python
def get_tenant():
tenant_name = request.headers.get("X-Tenant-Name")  # Example: Tenant from request header
if not tenant_name:
return None

    tenant = Tenant.query.filter_by(name=tenant_name).first()
    return tenant

@app.before_request
def set_tenant():
tenant = get_tenant()
if tenant:
app.config["SQLALCHEMY_DATABASE_URI"] = tenant.database_url
g.tenant = tenant
```

---

## **Multi-Tenancy Strategies**

### **1️⃣ Database-per-Tenant**

Each tenant has its own **separate database**. The application dynamically switches the database connection based on the tenant.

✅ **Strong data isolation**  
✅ **Easier tenant migration**  
❌ **High resource usage**

#### **Implementation**

```python
def switch_database(tenant):
if tenant:
app.config["SQLALCHEMY_DATABASE_URI"] = tenant.database_url
db.engine.dispose()  # Reset connection pool
```

When a request arrives, the database connection switches dynamically based on the tenant.

Example request:

```http
GET /dashboard
Host: app.example.com
X-Tenant-Name: "company1"
```

---

### **2️⃣ Schema-Based Multi-Tenancy**

Each tenant has a separate **schema within the same database**, keeping data isolated at the **schema level**.

✅ **Better resource utilization**  
✅ **Easy to manage backups**  
❌ **Schema migration complexity**

#### **Creating a Schema for Each Tenant**

```python
from sqlalchemy import text

def create_tenant_schema(tenant_name):
schema_name = f"tenant_{tenant_name}"
db.session.execute(text(f"CREATE SCHEMA IF NOT EXISTS {schema_name}"))
db.session.commit()
```

#### **Configuring Flask-SQLAlchemy to Use Schema**

```python
from sqlalchemy.schema import MetaData

def get_schema():
if g.tenant:
return f"tenant_{g.tenant.name}"
return "public"

metadata = MetaData(schema=get_schema())
db = SQLAlchemy(app, metadata=metadata)
```

Each request uses the correct schema dynamically.

---

### **3️⃣ Row-Level Multi-Tenancy**

All tenants share the **same database and tables**, but their data is separated using a **tenant_id column**.

✅ **Highly efficient for small tenants**  
✅ **Easy migrations**  
❌ **Risk of cross-tenant data leaks**

#### **Modifying Models for Tenant Filtering**

```python
class User(db.Model):
id = db.Column(db.Integer, primary_key=True)
tenant_id = db.Column(db.Integer, db.ForeignKey("tenant.id"), nullable=False)
name = db.Column(db.String(100), nullable=False)

@app.before_request
def enforce_tenant():
if g.tenant:
db.session.query(User).filter_by(tenant_id=g.tenant.id)
```

Every query will **automatically filter by tenant_id**.

---

## **Best Practices for Multi-Tenant Flask Apps**

✔ **Choose the right approach** based on your scalability needs.  
✔ **Use connection pooling** for database efficiency.  
✔ **Encrypt tenant-specific data** for security.  
✔ **Automate tenant onboarding** with schema creation and database provisioning.  
✔ **Implement robust logging and monitoring** for tenant performance tracking.

---

## **Conclusion**

By implementing **multi-tenancy in Flask**, you can build **scalable, secure, and cost-efficient** SaaS applications. Choose a **strategy** that best fits your **business and technical needs**:

✅ **Database-per-tenant** → Best for large-scale, isolated tenants.  
✅ **Schema-based multi-tenancy** → Best for medium-scale apps.  
✅ **Row-level multi-tenancy** → Best for smaller apps with shared data models.

🚀 **Start implementing multi-tenancy in your Flask applications today!**

---

**Do you have any questions? Drop them in the comments below!**  
