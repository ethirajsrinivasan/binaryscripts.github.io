---
layout: post
title: Deploying Ruby on Rails with Docker and Kubernetes for Scalable Applications
subtitle: Learn how to containerize and orchestrate your Ruby on Rails application using Docker and Kubernetes
categories: Ruby on Rails
tags: ["Rails", "Docker", "Kubernetes", "DevOps", "Containers"]
excerpt: A step-by-step guide on using Docker to containerize a Ruby on Rails application and deploying it on Kubernetes for scalability and resilience.
---

#### **Introduction**
Docker and Kubernetes have revolutionized **application deployment and scalability**.  
By containerizing Ruby on Rails applications with Docker and deploying them on Kubernetes, we can achieve:

- **Scalability**: Easily scale services up or down based on demand.
- **Portability**: Run the same containerized app in development, testing, and production.
- **Resilience**: Kubernetes ensures high availability and automated recovery.

In this guide, we will **dockerize** a Rails app and deploy it on a **Kubernetes cluster**.

---

#### **Step 1: Dockerizing a Ruby on Rails Application**
Docker helps package an application along with its dependencies.

##### **1.1 Create a Dockerfile**
Inside your Rails project, create a `Dockerfile`:

```dockerfile
# Use official Ruby image
FROM ruby:3.2.2

# Set working directory
WORKDIR /app

# Install dependencies
COPY Gemfile Gemfile.lock ./
RUN bundle install

# Copy the application code
COPY . .

# Precompile assets (optional)
RUN RAILS_ENV=production bundle exec rake assets:precompile

# Expose port 3000
EXPOSE 3000

# Start the Rails server
CMD ["rails", "server", "-b", "0.0.0.0"]
```

##### **1.2 Create a `.dockerignore` File**
Exclude unnecessary files:

```
.git
log/
tmp/
node_modules/
```

##### **1.3 Build and Run the Docker Container**
```sh
docker build -t my-rails-app .
docker run -p 3000:3000 my-rails-app
```

---

#### **Step 2: Running Rails with Docker Compose**
Docker Compose simplifies running **multi-container** applications.

##### **2.1 Create a `docker-compose.yml` File**
```yml
version: '3'
services:
db:
image: postgres:15
environment:
POSTGRES_USER: user
POSTGRES_PASSWORD: password
volumes:
- db_data:/var/lib/postgresql/data

web:
build: .
command: ["rails", "server", "-b", "0.0.0.0"]
ports:
- "3000:3000"
depends_on:
- db

volumes:
db_data:
```

##### **2.2 Start the Services**
```sh
docker-compose up --build
```

---

#### **Step 3: Deploying to Kubernetes**
Kubernetes manages containers across multiple servers.

##### **3.1 Create a Kubernetes Deployment**
Create a file `rails-deployment.yaml`:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
name: rails-app
spec:
replicas: 3
selector:
matchLabels:
app: rails
template:
metadata:
labels:
app: rails
spec:
containers:
- name: rails
image: my-rails-app:latest
ports:
- containerPort: 3000
```

##### **3.2 Create a Kubernetes Service**
```yml
apiVersion: v1
kind: Service
metadata:
name: rails-service
spec:
selector:
app: rails
ports:
- protocol: TCP
port: 80
targetPort: 3000
type: LoadBalancer
```

##### **3.3 Deploy to Kubernetes**
```sh
kubectl apply -f rails-deployment.yaml
kubectl apply -f rails-service.yaml
```

---

#### **Step 4: Scaling the Application**
Increase the number of instances:

```sh
kubectl scale deployment rails-app --replicas=5
```

Monitor running pods:

```sh
kubectl get pods
```

---

#### **Step 5: Automating Deployment with CI/CD**
Use **GitHub Actions** or **GitLab CI/CD** to automate deployments.  
Example GitHub Actions workflow:

```yml
name: Deploy Rails to Kubernetes

on:
push:
branches:
- main

jobs:
deploy:
runs-on: ubuntu-latest
steps:
- uses: actions/checkout@v3
- name: Build Docker Image
run: docker build -t my-rails-app .
- name: Push to Docker Hub
run: docker push my-rails-app
- name: Deploy to Kubernetes
run: kubectl apply -f rails-deployment.yaml
```

---

#### **Conclusion**
Using Docker and Kubernetes with Rails provides **scalability, resilience, and automation**.
- **Docker** makes development and deployment consistent.
- **Kubernetes** ensures high availability and load balancing.
- **CI/CD** automates the entire deployment pipeline.

By following this guide, you can build a **highly scalable Rails application** ready for production! 🚀
