---
layout: post
title: "Integrating Flask with Front-End Frameworks like Vue.js and React"
subtitle: "Build full-stack applications by seamlessly connecting Flask with Vue.js and React"
categories: Flask
tags: ["Flask", "Vue.js", "React", "Full Stack", "API", "JavaScript", "Python", "Frontend"]
excerpt: "Learn how to integrate Flask with modern front-end frameworks like Vue.js and React to build seamless full-stack applications. Explore API design, CORS handling, authentication, and deployment strategies."
---
Flask is a powerful backend framework for building APIs, but integrating it with **modern front-end frameworks like Vue.js and React** can be challenging. This guide covers **best practices for API design, handling CORS, authentication, and deployment strategies** to create a seamless full-stack experience.

## Setting Up the Flask Backend

#### Creating a Simple Flask API

First, let's create a **Flask API** that serves data to the front-end.

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/api/data', methods=['GET'])
def get_data():
return jsonify({"message": "Hello from Flask!"})

if __name__ == '__main__':
app.run(debug=True)
```

This simple API returns a JSON response that can be consumed by Vue.js or React.

## Handling CORS in Flask

Since front-end applications run on a different port (e.g., Vue.js at **localhost:8080** and Flask at **localhost:5000**), enabling **CORS (Cross-Origin Resource Sharing)** is essential.

```python
from flask_cors import CORS

app = Flask(__name__)
CORS(app)
```

This allows Vue.js or React to make requests to the Flask backend without CORS errors.

## Integrating Flask with Vue.js

#### Fetching API Data in Vue.js

Inside your Vue.js component, fetch data from the Flask API using **Axios**.

```vue
<template>
  <div>
    <h2>{{ message }}</h2>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      message: ''
    };
  },
  mounted() {
    axios.get('http://localhost:5000/api/data')
      .then(response => {
        this.message = response.data.message;
      })
      .catch(error => console.error(error));
  }
};
</script>
```

This retrieves data from Flask and displays it in the Vue component.

## Integrating Flask with React

#### Fetching API Data in React

Inside your React component, use **fetch()** or Axios to request data from the Flask API.

```jsx
import React, { useState, useEffect } from "react";
import axios from "axios";

function App() {
const [message, setMessage] = useState("");

useEffect(() => {
axios.get("http://localhost:5000/api/data")
.then(response => setMessage(response.data.message))
.catch(error => console.error(error));
}, []);

return <h2>{message}</h2>;
}

export default App;
```

This React component fetches data from the Flask backend and displays it.

## Authentication with JWT

#### Securing API Routes in Flask

To protect API routes, implement **JWT authentication** in Flask.

```python
from flask import request, jsonify
import jwt
import datetime

SECRET_KEY = "your_secret_key"

@app.route('/api/protected', methods=['GET'])
def protected():
token = request.headers.get("Authorization")

    if not token:
        return jsonify({"message": "Token is missing"}), 401

    try:
        jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return jsonify({"message": "Secure data"})
    except:
        return jsonify({"message": "Invalid token"}), 403
```

#### Sending JWT in React/Vue Requests

After login, store the JWT token and send it in **Axios headers** for secure API requests.

```js
axios.get("http://localhost:5000/api/protected", {
headers: { Authorization: "Bearer " + localStorage.getItem("token") }
});
```

This ensures secure communication between the front-end and Flask API.

## Deploying Flask with Vue.js or React

#### Building and Serving Front-End with Flask

To serve the Vue.js or React build inside Flask, place the compiled front-end files in **Flask’s static folder**.

```python
from flask import Flask, send_from_directory

app = Flask(__name__, static_folder="frontend/dist")

@app.route("/")
def serve_frontend():
return send_from_directory("frontend/dist", "index.html")
```

This serves the Vue.js or React front-end directly from Flask.

## Conclusion

By following these steps, you can seamlessly integrate Flask with **Vue.js and React** to build a full-stack application. From **API design, CORS handling, authentication, to deployment**, this guide provides a robust foundation for your next project.

🚀 **Start building your full-stack Flask app today!**
