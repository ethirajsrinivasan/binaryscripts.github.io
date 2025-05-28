---
layout: post
title: CI CD with Docker Automating Container Builds Testing and Deployments
subtitle: Streamline your software delivery pipeline by integrating Docker with CI/CD workflows
categories: Docker
tags: [Docker, CI/CD, DevOps, Jenkins, GitHub Actions, Kubernetes, Automation, Containers]
excerpt: Learn how to implement CI/CD pipelines with Docker to automate container builds, testing, and deployments. Explore tools like Jenkins, GitHub Actions, and Kubernetes for a robust DevOps workflow.
---
**Continuous Integration and Continuous Deployment (CI/CD)** are essential practices for modern DevOps teams aiming to deliver software faster and more reliably. When combined with **Docker**, CI/CD pipelines can package applications as containers that are easy to test, deploy, and scale across environments.

In this post, we’ll explore how to set up CI/CD pipelines for Docker-based applications, including automated **container builds**, **testing**, and **deployments** using tools like **Jenkins**, **GitHub Actions**, and **Kubernetes**.

---

#### Why Use Docker in CI/CD?

Docker provides consistent environments that solve the "works on my machine" problem. Benefits in a CI/CD pipeline include:

- **Portable, reproducible builds**
- **Environment consistency** from development to production
- **Simplified dependency management**
- **Faster testing and deployment cycles**
- Easy integration with orchestration tools like Kubernetes

---

#### Typical CI/CD Pipeline with Docker

```
[Code Commit] → [CI Server (Jenkins/GitHub Actions)]  
→ [Docker Build]  
→ [Automated Tests]  
→ [Push to Registry]  
→ [Deploy to Kubernetes/VM/Cloud]
```

Each step can be automated to trigger on code changes, reducing manual intervention and speeding up feedback loops.

---

#### Step 1: Dockerfile Best Practices

Your CI/CD pipeline starts with a well-structured `Dockerfile`.

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
CMD ["node", "server.js"]
EXPOSE 3000
```

Tips:
- Use multi-stage builds to reduce image size
- Pin base image versions
- Avoid copying unnecessary files (`.dockerignore`)

---

#### Step 2: CI Integration with GitHub Actions

Example `.github/workflows/docker-ci.yml` for Node.js:

```yml
name: CI with Docker

on:
push:
branches: [main]

jobs:
build-and-test:
runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Build Docker Image
      run: docker build -t my-app:latest .

    - name: Run Unit Tests
      run: docker run --rm my-app:latest npm test
```

You can extend this to include image tagging, security scanning, and pushing to registries.

---

#### Step 3: Pushing to Docker Registry

Use GitHub Actions or Jenkins to push your image:

```yml
- name: Login to Docker Hub
  run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin

- name: Tag and Push
  run: |
  docker tag my-app:latest mydockerhubuser/my-app:1.0.${{ github.run_number }}
  docker push mydockerhubuser/my-app:1.0.${{ github.run_number }}
  ```

You can also push to **Amazon ECR**, **Google Container Registry**, or **Azure Container Registry**.

---

#### Step 4: Deploying with Kubernetes

Once your Docker image is built and pushed, deploy using `kubectl` or Helm.

```yml
- name: Deploy to Kubernetes
  run: |
  kubectl set image deployment/my-app-deployment my-app=mydockerhubuser/my-app:1.0.${{ github.run_number }} --record
  ```

Integrate Kubernetes secrets and config maps for environment-specific values.

---

#### Optional: Jenkins CI with Docker

A basic Jenkins pipeline (Jenkinsfile):

```groovy
pipeline {
agent any

stages {
stage('Build') {
steps {
sh 'docker build -t my-app .'
}
}

    stage('Test') {
      steps {
        sh 'docker run --rm my-app npm test'
      }
    }

    stage('Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            echo $PASS | docker login -u $USER --password-stdin
            docker tag my-app mydockerhubuser/my-app:latest
            docker push mydockerhubuser/my-app:latest
          '''
        }
      }
    }
}
}
```

---

#### Best Practices

- Use **.dockerignore** to exclude test files and secrets
- Automate **security scanning** using tools like Trivy or Docker Scout
- Use **image tags** with version and commit hash (e.g., `app:1.2.3-abc123`)
- Implement **rollback strategies** in deployment
- Cache dependencies (like npm or pip) across builds for faster CI

---

#### Conclusion

By combining Docker with CI/CD tools like GitHub Actions or Jenkins, teams can **automate the entire delivery pipeline**, from building containers to running tests and deploying into production.

With proper configuration and best practices, you’ll ensure **reliable**, **repeatable**, and **secure software delivery** across development, staging, and production environments.

Start small, iterate fast, and let Docker-powered CI/CD take your DevOps game to the next level.
