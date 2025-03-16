---
layout: post
title: "Using Flask for Server-Side Rendering with Jinja2 Templates"
subtitle: "Leverage Flask and Jinja2 for efficient server-side rendering in web applications"
categories: Flask
tags: ["Flask", "Jinja2", "Server-Side Rendering", "Python", "Web Development", "Templates"]
excerpt: "Learn how to use Flask with Jinja2 templates for server-side rendering, improving SEO, performance, and dynamic content delivery in web applications."
---

## Introduction

**Flask**, a lightweight Python web framework, includes **Jinja2**, a powerful templating engine that enables **server-side rendering (SSR)**. Unlike client-side rendering, **SSR improves SEO, performance, and accessibility**, making it a preferred choice for many web applications.

This guide explores **setting up Flask with Jinja2, rendering dynamic content, using template inheritance, handling forms, and deploying a fully functional SSR web application**.

## Setting Up Flask with Jinja2

To begin, install Flask:

```sh
pip install flask
```

#### Basic Flask App with Jinja2

Create a simple Flask app that renders an **HTML page using Jinja2 templates**.

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def home():
return render_template("index.html", title="Home Page", message="Welcome to Flask SSR!")

if __name__ == "__main__":
app.run(debug=True)
```

## Creating Jinja2 Templates

Flask looks for templates inside the `templates/` folder. Create `templates/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }}</title>
</head>
<body>
    <h1>{{ message }}</h1>
</body>
</html>
```  

Here, `{{ title }}` and `{{ message }}` are Jinja2 placeholders dynamically populated by Flask.

## Template Inheritance in Jinja2

To maintain a **consistent layout**, use **Jinja2 template inheritance**.

### Creating a Base Template

Create `templates/base.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Flask App{% endblock %}</title>
</head>
<body>
    <header>
        <h1>My Flask App</h1>
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
</body>
</html>
```  

### Extending the Base Template

Now, extend it in `index.html`:

```html
{% extends "base.html" %}

{% block title %}Home - Flask SSR{% endblock %}

{% block content %}
<h2>Welcome to Flask Server-Side Rendering</h2>
<p>This page is rendered dynamically using Jinja2.</p>
{% endblock %}
```

## Handling Forms in Flask

### Creating a Simple Form

Forms in Flask allow users to submit data to the server.

```html
<form action="/submit" method="POST">
    <input type="text" name="username" placeholder="Enter your name" required>
    <button type="submit">Submit</button>
</form>
```  

### Handling Form Submission

Modify `app.py` to process the form data:

```python
from flask import request

@app.route("/submit", methods=["POST"])
def submit():
username = request.form.get("username")
return render_template("index.html", title="Submission", message=f"Hello, {username}!")
```

## Rendering Dynamic Content

Jinja2 supports **loops** and **conditional statements** to render dynamic data.

### Looping Over a List

```html
<ul>
    {% for item in items %}
        <li>{{ item }}</li>
    {% endfor %}
</ul>
```  

### Using Conditionals

```html
{% if user %}
<p>Welcome back, {{ user }}!</p>
{% else %}
<p>Please log in.</p>
{% endif %}
```

## Deploying a Flask SSR Application

### Running the Flask App

Execute the following command:

```sh
python app.py
```

### Deploying with Gunicorn

For production deployment, use **Gunicorn**:

```sh
pip install gunicorn
gunicorn -w 4 app:app
```

## Conclusion

Flask’s **server-side rendering with Jinja2** enables **faster, SEO-friendly, and dynamic web applications**. By leveraging **template inheritance, form handling, and deployment techniques**, you can build powerful web applications efficiently.

🚀 **Start building your Flask SSR application today!**  
