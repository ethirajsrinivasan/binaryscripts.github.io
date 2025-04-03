---
layout: post
title: "Building CI/CD Pipelines for Flask Applications"
subtitle: "Automate deployment and testing of Flask applications with CI/CD pipelines"
categories: Flask
tags: ["Flask", "CI/CD", "DevOps", "GitHub Actions", "Docker", "Kubernetes"]
excerpt: "Learn how to set up a CI/CD pipeline for Flask applications using GitHub Actions, Docker, and Kubernetes to automate testing and deployment."
---



Continuous Integration and Continuous Deployment (**CI/CD**) play a crucial role in modern software development. Automating the testing, building, and deployment process ensures **faster delivery, fewer errors, and improved reliability**.

In this guide, we will explore **how to build a CI/CD pipeline for Flask applications** using:

✅ **GitHub Actions** for automated builds and testing  
✅ **Docker** for containerization  
✅ **Kubernetes** for deployment  
✅ **NGINX/Gunicorn** for production-ready hosting

---

## Setting Up Flask for CI/CD

Before setting up the pipeline, ensure your Flask project has:

- A **GitHub repository**
- A **requirements.txt** file for dependencies
- A **Dockerfile** for containerization
- A **test suite** using `pytest`

### Step 1: Create a Basic Flask Application

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
return "Hello, CI/CD Pipeline!"

if __name__ == "__main__":
app.run(host="0.0.0.0", port=5000)
```

### Step 2: Add Dependencies

Create a **requirements.txt** file:

```txt
Flask
pytest
```

---

## Writing Unit Tests

Automated testing is crucial in CI/CD pipelines. Let’s write a simple **pytest** test case.

Create a `test_app.py` file:

```python
import pytest
from app import app

@pytest.fixture
def client():
app.testing = True
return app.test_client()

def test_home(client):
response = client.get("/")
assert response.status_code == 200
assert b"Hello, CI/CD Pipeline!" in response.data
```

Run tests locally:

```sh
pytest
```

---

## Dockerizing the Flask Application

### Step 3: Create a Dockerfile

```dockerfile
# Use official Python image
FROM python:3.9

# Set the working directory
WORKDIR /app

# Copy application files
COPY . /app

# Install dependencies
RUN pip install -r requirements.txt

# Expose port 5000
EXPOSE 5000

# Start Flask application
CMD ["python", "app.py"]
```

Build and run the Docker container:

```sh
docker build -t flask-ci-cd .
docker run -p 5000:5000 flask-ci-cd
```

---

## Setting Up GitHub Actions for CI

GitHub Actions automates testing and deployment.

### Step 4: Create a `.github/workflows/ci.yml` File

```yml
name: Flask CI Pipeline

on:
push:
branches:
- main
pull_request:
branches:
- main

jobs:
test:
runs-on: ubuntu-latest
steps:
- name: Checkout Code
uses: actions/checkout@v3

      - name: Set Up Python
        uses: actions/setup-python@v3
        with:
          python-version: "3.9"

      - name: Install Dependencies
        run: |
          pip install -r requirements.txt

      - name: Run Tests
        run: pytest
```

This workflow **automatically runs tests** on every push or pull request.

---

## Deploying with Docker and Kubernetes

### Step 5: Create a Kubernetes Deployment

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
name: flask-app
spec:
replicas: 2
selector:
matchLabels:
app: flask
template:
metadata:
labels:
app: flask
spec:
containers:
- name: flask-container
image: your-dockerhub-username/flask-ci-cd:latest
ports:
- containerPort: 5000
```

Apply the deployment:

```sh
kubectl apply -f deployment.yml
```

---

## Automating Deployment with GitHub Actions

### Step 6: Extend GitHub Actions for Deployment

Modify `.github/workflows/ci.yml` to include **Docker build & push**:

```yml
build-and-deploy:
needs: test
runs-on: ubuntu-latest
steps:
- name: Checkout Code
uses: actions/checkout@v3

      - name: Set Up Docker
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker build -t your-dockerhub-username/flask-ci-cd .
          docker tag your-dockerhub-username/flask-ci-cd your-dockerhub-username/flask-ci-cd:latest
          docker push your-dockerhub-username/flask-ci-cd:latest
```

### Step 7: Deploy to Kubernetes

```yml
- name: Deploy to Kubernetes
run: |
echo "${{ secrets.KUBECONFIG }}" | base64 --decode > kubeconfig
kubectl --kubeconfig=kubeconfig apply -f deployment.yml
```

This ensures **automatic deployment** to Kubernetes after successful tests.

---

## Conclusion

🚀 **Key Takeaways:**

✅ **Automated Testing** using GitHub Actions  
✅ **Containerization** with Docker  
✅ **Automated Deployments** with Kubernetes

By implementing **CI/CD**, you can ensure smooth, reliable, and automated deployments of your Flask applications.

🔗 **Next Steps:** Integrate with **Helm Charts**, set up **CDNs**, or explore **serverless deployments**!

---

💡 **Did you find this helpful?** Share your thoughts in the comments! 🚀  
