---
layout: post
title: "Designing RESTful APIs in Flask with Advanced Query Parameters"
subtitle: "Enhance your Flask APIs with powerful query parameters for filtering, pagination, and sorting"
categories: Flask
tags: ["Flask", "REST API", "Query Parameters", "Filtering", "Pagination", "Sorting"]
excerpt: "Learn how to design RESTful APIs in Flask with advanced query parameters for efficient data retrieval, including filtering, pagination, and sorting techniques."
---



Designing **RESTful APIs** in Flask goes beyond basic route handling. To create **scalable, user-friendly APIs**, implementing **advanced query parameters** is essential. This guide will cover how to **filter, paginate, and sort** API results efficiently using **Flask and SQLAlchemy**.

## Setting Up a Flask API

### Installing Dependencies

Ensure you have Flask and SQLAlchemy installed:

```sh
pip install flask flask_sqlalchemy flask_marshmallow marshmallow-sqlalchemy
```

- **flask_sqlalchemy**: ORM for handling database interactions.
- **flask_marshmallow**: Serializes data into JSON.

### Creating a Flask App with SQLAlchemy

```python
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_marshmallow import Marshmallow

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///products.db"
db = SQLAlchemy(app)
ma = Marshmallow(app)

class Product(db.Model):
id = db.Column(db.Integer, primary_key=True)
name = db.Column(db.String(100), nullable=False)
category = db.Column(db.String(50), nullable=False)
price = db.Column(db.Float, nullable=False)
stock = db.Column(db.Integer, nullable=False)

class ProductSchema(ma.SQLAlchemyAutoSchema):
class Meta:
model = Product

product_schema = ProductSchema()
products_schema = ProductSchema(many=True)

@app.route("/products", methods=["GET"])
def get_products():
products = Product.query.all()
return jsonify(products_schema.dump(products))

if __name__ == "__main__":
app.run(debug=True)
```

This sets up a **basic API** for retrieving products from an SQLite database.

---

## Implementing Advanced Query Parameters

### 1. Filtering

Enable filtering by query parameters like **category** and **price range**:

```python
@app.route("/products", methods=["GET"])
def get_products():
query = Product.query

    category = request.args.get("category")
    if category:
        query = query.filter(Product.category == category)

    min_price = request.args.get("min_price", type=float)
    max_price = request.args.get("max_price", type=float)
    if min_price is not None:
        query = query.filter(Product.price >= min_price)
    if max_price is not None:
        query = query.filter(Product.price <= max_price)

    products = query.all()
    return jsonify(products_schema.dump(products))
```

### Example Requests:

- `/products?category=electronics` → Returns only **electronics**.
- `/products?min_price=100&max_price=500` → Filters products within a price range.

---

### 2. Pagination

To **limit results** and **optimize API performance**, implement **pagination**:

```python
@app.route("/products", methods=["GET"])
def get_products():
query = Product.query

    page = request.args.get("page", 1, type=int)
    per_page = request.args.get("per_page", 10, type=int)
    
    paginated_query = query.paginate(page=page, per_page=per_page, error_out=False)
    
    return jsonify({
        "total": paginated_query.total,
        "page": page,
        "per_page": per_page,
        "products": products_schema.dump(paginated_query.items)
    })
```

### Example Requests:

- `/products?page=2&per_page=5` → Fetches **page 2**, **5 products per page**.

---

### 3. Sorting

Enable sorting by **price, name, or stock**:

```python
@app.route("/products", methods=["GET"])
def get_products():
query = Product.query

    sort_by = request.args.get("sort_by", "id")
    order = request.args.get("order", "asc")

    if sort_by in ["price", "name", "stock"]:
        column = getattr(Product, sort_by)
        query = query.order_by(column.asc() if order == "asc" else column.desc())

    products = query.all()
    return jsonify(products_schema.dump(products))
```

### Example Requests:

- `/products?sort_by=price&order=desc` → Sorts by **price in descending order**.
- `/products?sort_by=name&order=asc` → Sorts by **name alphabetically**.

---

## Combining Filtering, Pagination, and Sorting

The final **optimized endpoint**:

```python
@app.route("/products", methods=["GET"])
def get_products():
query = Product.query

    # Filtering
    category = request.args.get("category")
    if category:
        query = query.filter(Product.category == category)

    min_price = request.args.get("min_price", type=float)
    max_price = request.args.get("max_price", type=float)
    if min_price is not None:
        query = query.filter(Product.price >= min_price)
    if max_price is not None:
        query = query.filter(Product.price <= max_price)

    # Sorting
    sort_by = request.args.get("sort_by", "id")
    order = request.args.get("order", "asc")
    if sort_by in ["price", "name", "stock"]:
        column = getattr(Product, sort_by)
        query = query.order_by(column.asc() if order == "asc" else column.desc())

    # Pagination
    page = request.args.get("page", 1, type=int)
    per_page = request.args.get("per_page", 10, type=int)
    paginated_query = query.paginate(page=page, per_page=per_page, error_out=False)

    return jsonify({
        "total": paginated_query.total,
        "page": page,
        "per_page": per_page,
        "products": products_schema.dump(paginated_query.items)
    })
```

### Example Request:

&#96;/products?category=electronics&min_price=100&sort_by=price&order=desc&page=1&per_page=5&#96;

- Filters by **category** and **min price**.
- Sorts by **price descending**.
- Fetches **page 1** with **5 results per page**.

---

## Best Practices for Query Parameters in REST APIs

1. **Use consistent parameter naming** (`min_price`, `sort_by`, `page`).
2. **Validate inputs** to avoid errors and SQL injection risks.
3. **Set reasonable defaults** for pagination and sorting.
4. **Return metadata** (`total records`, `page number`, `items per page`).

---

## Conclusion

By integrating **advanced query parameters**, your Flask API becomes **more flexible and efficient**, allowing users to filter, paginate, and sort results dynamically.

🚀 **Start designing powerful RESTful APIs in Flask today!**  
