---
layout: post
title: "Custom Error Pages and Exception Handling in Flask"
subtitle: "Enhance user experience by handling errors effectively in Flask applications"
categories: Flask
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: ["Flask", "Error Handling", "Custom Error Pages", "HTTP Exceptions", "Logging"]
excerpt: "Learn how to create custom error pages and handle exceptions in Flask applications. Improve UX by designing user-friendly error responses and logging critical errors."
---
Handling errors properly is **crucial** for any web application. Flask provides flexible mechanisms to manage **exceptions** and display **custom error pages**. In this guide, we will explore:

- Handling built-in HTTP errors
- Creating custom error pages
- Managing exceptions with `@app.errorhandler`
- Logging errors for debugging and monitoring

## Default Flask Error Handling

By default, Flask provides generic error pages for **common HTTP errors**, such as:

- `404 Not Found`
- `403 Forbidden`
- `500 Internal Server Error`

If no custom handling is implemented, Flask returns default error responses like:

```json
{
"error": "Not Found",
"status": 404
}
```

## Creating Custom Error Pages

### Registering Error Handlers

Use `@app.errorhandler` to define **custom error responses**:

```python
from flask import Flask, render_template, jsonify

app = Flask(__name__)

@app.errorhandler(404)
def not_found_error(error):
return render_template("404.html"), 404

@app.errorhandler(500)
def internal_error(error):
return render_template("500.html"), 500
```

### Designing HTML Error Pages

Create a **404.html** template inside the `templates/` folder:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Page Not Found</title>
</head>
<body>
    <h1>Oops! 404 Error</h1>
    <p>The page you're looking for doesn't exist.</p>
    <a href="/">Go Back Home</a>
</body>
</html>
```  

Now, whenever a **404 error** occurs, users see this custom page instead of a generic error.

## Handling Exceptions Gracefully

Flask allows handling **unhandled exceptions** globally using `@app.errorhandler(Exception)`.

```python
@app.errorhandler(Exception)
def handle_exception(error):
response = {
"error": str(error),
"status": 500
}
return jsonify(response), 500
```

This ensures that **all unhandled errors** return a structured JSON response instead of a full traceback.

## Customizing API Error Responses

For **REST APIs**, return JSON instead of HTML.

```python
@app.errorhandler(404)
def not_found_api(error):
return jsonify({"error": "Resource not found", "status": 404}), 404

@app.errorhandler(400)
def bad_request(error):
return jsonify({"error": "Bad request", "status": 400}), 400
```

When an API request fails, it returns:

```json
{
"error": "Resource not found",
"status": 404
}
```

## Logging Errors for Debugging

To track errors, configure **logging**:

```python
import logging

# Configure logging
logging.basicConfig(filename="error.log", level=logging.ERROR)

@app.errorhandler(500)
def internal_error(error):
app.logger.error(f"Server Error: {error}")
return jsonify({"error": "Internal Server Error"}), 500
```

Now, errors are saved in `error.log` for debugging.

## Raising Custom Exceptions

Define **custom exception classes**:

```python
class InvalidUsage(Exception):
def __init__(self, message, status_code=400):
super().__init__(message)
self.message = message
self.status_code = status_code

@app.errorhandler(InvalidUsage)
def handle_invalid_usage(error):
return jsonify({"error": error.message}), error.status_code

@app.route("/raise-error")
def raise_error():
raise InvalidUsage("This is a custom error", 422)
```

Now, accessing `/raise-error` triggers:

```json
{
"error": "This is a custom error"
}
```

## Conclusion

Implementing **custom error handling** in Flask:  
✅ Enhances **user experience** with friendly error pages  
✅ Improves **API responses** with structured error messages  
✅ Enables **better debugging** through logging

Start building **robust Flask applications** with proper error handling today! 🚀  
