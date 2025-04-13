---
layout: post
title: "Flask and Elasticsearch Implementing Advanced Search"
subtitle: "Enhance your Flask applications with powerful search capabilities using Elasticsearch"
categories: Flask
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: ["Flask", "Elasticsearch", "Search", "Full-Text Search", "Indexing", "Big Data"]
excerpt: "Learn how to integrate Elasticsearch with Flask to implement advanced search functionalities. This guide covers indexing, querying, and optimizing search performance."
---
Search functionality is a **critical feature** in modern web applications. **Elasticsearch**, a powerful **full-text search engine**, enables fast and scalable search capabilities. In this guide, we will integrate **Flask with Elasticsearch** to build an **advanced search system** with **indexing, querying, and filtering**.

## Setting Up Elasticsearch

### Installing Elasticsearch

First, install **Elasticsearch** using Docker:

```bash
docker run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:8.4.0
```

Verify the installation:

```bash
curl http://localhost:9200
```

If Elasticsearch is running, it will return a JSON response with version details.

## Integrating Flask with Elasticsearch

### Installing Dependencies

In your Flask project, install the required packages:

```bash
pip install Flask elasticsearch requests
```

### Configuring Elasticsearch in Flask

Create a Flask app and connect it to Elasticsearch:

```python
from flask import Flask, request, jsonify
from elasticsearch import Elasticsearch

app = Flask(__name__)
es = Elasticsearch("http://localhost:9200")

@app.route("/")
def home():
return "Flask & Elasticsearch API is running!"

if __name__ == "__main__":
app.run(debug=True)
```

## Indexing Data in Elasticsearch

### Creating an Index

An **index** in Elasticsearch stores and organizes documents for search.

```python
index_name = "products"

# Define index mapping
index_mapping = {
"settings": {"number_of_shards": 1},
"mappings": {
"properties": {
"name": {"type": "text"},
"description": {"type": "text"},
"price": {"type": "float"},
"category": {"type": "keyword"}
}
}
}

# Create the index
es.indices.create(index=index_name, body=index_mapping, ignore=400)
```

### Adding Data to the Index

Insert sample data into Elasticsearch:

```python
def add_product(product_id, name, description, price, category):
doc = {
"name": name,
"description": description,
"price": price,
"category": category
}
es.index(index=index_name, id=product_id, body=doc)

# Example data
add_product(1, "Laptop", "High-performance laptop", 1200, "Electronics")
add_product(2, "Smartphone", "Latest model smartphone", 800, "Electronics")
```

## Implementing Advanced Search

### Basic Full-Text Search

A simple **search query** to find products by name or description:

```python
@app.route("/search", methods=["GET"])
def search():
query = request.args.get("q")
search_query = {
"query": {
"multi_match": {
"query": query,
"fields": ["name", "description"]
}
}
}
results = es.search(index=index_name, body=search_query)
return jsonify(results["hits"]["hits"])
```

Access the API with:

```bash
curl "http://127.0.0.1:5000/search?q=laptop"
```

### Filtering Results

To filter by **category** and **price range**:

```python
@app.route("/filter", methods=["GET"])
def filter_results():
category = request.args.get("category")
min_price = float(request.args.get("min_price", 0))
max_price = float(request.args.get("max_price", 10000))

    filter_query = {
        "query": {
            "bool": {
                "must": [
                    {"term": {"category": category}}
                ],
                "filter": [
                    {"range": {"price": {"gte": min_price, "lte": max_price}}}
                ]
            }
        }
    }
    results = es.search(index=index_name, body=filter_query)
    return jsonify(results["hits"]["hits"])
```

Access the API with:

```bash
curl "http://127.0.0.1:5000/filter?category=Electronics&min_price=500&max_price=1500"
```

## Optimizing Search Performance

### Using Analyzers for Better Text Search

Define a **custom analyzer** for improved search accuracy:

```python
index_mapping["settings"]["analysis"] = {
"analyzer": {
"custom_analyzer": {
"type": "standard",
"stopwords": "_english_"
}
}
}
```

### Implementing Pagination

To handle **large search results**, add pagination:

```python
@app.route("/search_paginated", methods=["GET"])
def search_paginated():
query = request.args.get("q")
page = int(request.args.get("page", 1))
size = int(request.args.get("size", 10))
search_query = {
"query": {
"multi_match": {
"query": query,
"fields": ["name", "description"]
}
},
"from": (page - 1) * size,
"size": size
}
results = es.search(index=index_name, body=search_query)
return jsonify(results["hits"]["hits"])
```

Access paginated results with:

```bash
curl "http://127.0.0.1:5000/search_paginated?q=laptop&page=1&size=5"
```

## Conclusion

By integrating **Elasticsearch with Flask**, you can build **powerful search features** that handle **full-text search, filtering, and pagination** efficiently. Optimizing queries and indexes ensures **fast performance and scalability**.

Start building **advanced search functionality** in your Flask applications today! 🚀  
