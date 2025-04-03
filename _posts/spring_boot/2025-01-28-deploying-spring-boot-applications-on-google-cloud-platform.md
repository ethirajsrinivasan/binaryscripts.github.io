---
layout: post
title: Deploying Spring Boot Applications on Google Cloud Platform
subtitle: Learn how to deploy and manage Spring Boot applications on GCP using App Engine, Cloud Run, and GKE
categories: Spring Boot
tags: [Spring Boot, GCP, Google Cloud, Deployment, App Engine, Cloud Run, Kubernetes, DevOps]
excerpt: Discover the complete process of deploying Spring Boot applications on Google Cloud Platform. This guide covers App Engine, Cloud Run, Kubernetes, and best practices for cloud-native deployment.
---



Deploying your **Spring Boot** applications to the cloud is a critical step toward scalability, resilience, and modern software delivery. **Google Cloud Platform (GCP)** offers a suite of services to host and manage Spring Boot apps — from serverless environments to fully containerized infrastructures.

In this guide, you’ll learn how to deploy Spring Boot applications using:
- **App Engine**
- **Cloud Run**
- **Google Kubernetes Engine (GKE)**

We’ll walk through configurations, deployment steps, and best practices for each method, so you can choose the best option for your workload.

---

#### Preparing Your Spring Boot App for Deployment

Make sure your application:
- Has an embedded server (Tomcat or Jetty)
- Exposes an HTTP port (usually `8080`)
- Can be packaged as a JAR or containerized as a Docker image

Build the JAR:

```bash
./mvnw clean package
```

Ensure the `application.properties` or `application.yml` is cloud-friendly:

```yml
server:
port: 8080
spring:
application:
name: my-spring-boot-app
```

---

#### Option 1: Deploying to App Engine (Standard or Flex)

**App Engine** is a fully managed platform for deploying Java apps without managing infrastructure.

**Step 1: Add App Engine Plugin**

```xml
<plugin>
<groupId>com.google.cloud.tools</groupId>
<artifactId>appengine-maven-plugin</artifactId>
<version>2.4.2</version>
</plugin>
```

**Step 2: Create `app.yaml`**

For **Flexible Environment** (supports Spring Boot JARs):

```yaml
runtime: java
env: flex
service: default
manual_scaling:
instances: 1
```

**Step 3: Deploy**

```bash
gcloud app deploy
```

**Pros:**
- No server management
- Auto-scaling support
- Built-in logging and monitoring

---

#### Option 2: Deploying to Cloud Run

**Cloud Run** runs containerized applications in a fully managed serverless environment.

**Step 1: Create a Dockerfile**

```Dockerfile
FROM openjdk:17-jdk-slim
VOLUME /tmp
COPY target/demo.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

**Step 2: Build and Push to Google Container Registry**

```bash
gcloud builds submit --tag gcr.io/YOUR_PROJECT_ID/spring-boot-app
```

**Step 3: Deploy to Cloud Run**

```bash
gcloud run deploy spring-boot-app \
--image gcr.io/YOUR_PROJECT_ID/spring-boot-app \
--platform managed \
--region us-central1 \
--allow-unauthenticated
```

**Pros:**
- Fast deployments
- Serverless scaling
- HTTP/2, TLS, and request-based pricing

---

#### Option 3: Deploying to GKE (Google Kubernetes Engine)

**GKE** offers a fully managed Kubernetes platform.

**Step 1: Containerize Your App**

Use the Dockerfile created earlier.

**Step 2: Push the Image**

```bash
docker tag demo gcr.io/YOUR_PROJECT_ID/demo
docker push gcr.io/YOUR_PROJECT_ID/demo
```

**Step 3: Create Kubernetes Deployment**

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
name: spring-boot-app
spec:
replicas: 2
selector:
matchLabels:
app: spring-boot-app
template:
metadata:
labels:
app: spring-boot-app
spec:
containers:
- name: app
image: gcr.io/YOUR_PROJECT_ID/demo
ports:
- containerPort: 8080
```

**Step 4: Expose the Service**

```yml
apiVersion: v1
kind: Service
metadata:
name: spring-boot-service
spec:
type: LoadBalancer
selector:
app: spring-boot-app
ports:
- port: 80
targetPort: 8080
```

**Pros:**
- Full control over environment
- Great for microservices
- Scales to complex workloads

---

#### Managing Environment Configuration

Use **ConfigMaps** and **Secrets** in Kubernetes, or **runtime environment variables** in App Engine and Cloud Run.

For Cloud Run:

```bash
gcloud run deploy --set-env-vars SPRING_PROFILES_ACTIVE=prod,LOG_LEVEL=DEBUG
```

Externalize `application.yml` using volume mounts or mounted secrets in GKE.

---

#### Monitoring and Logging

GCP integrates with **Cloud Monitoring** and **Cloud Logging** out of the box.

For Spring Boot apps with Actuator:

- Add `spring-boot-starter-actuator`
- Enable `/actuator/metrics`, `/actuator/health`
- View logs in **Cloud Console > Logs Explorer**

You can export metrics to Prometheus using Micrometer and visualize in Grafana.

---

#### Best Practices

- Use **profiles** for dev, staging, and production environments
- Configure **readiness and liveness probes** in GKE
- Enable **autoscaling** in Cloud Run and GKE
- Use **Cloud Build** or **GitHub Actions** for CI/CD
- Secure your endpoints with **IAM roles** and **Cloud Armor**

---

#### Conclusion

Google Cloud Platform provides multiple ways to deploy Spring Boot applications — from serverless simplicity with Cloud Run to container orchestration with GKE.

Each method offers unique strengths, and choosing the right one depends on your scale, complexity, and team expertise. With the right deployment strategy, your Spring Boot app can be **resilient, observable, and ready to scale** on Google Cloud.
