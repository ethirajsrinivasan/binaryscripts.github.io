---
layout: post
title: Using Spring Boot with Docker and Kubernetes for Scalable Deployments
subtitle: Containerize and orchestrate your Spring Boot applications with Docker and Kubernetes for enterprise-scale deployment
categories: SpringBoot
tags: [Java, Spring Boot, Docker, Kubernetes, Microservices, DevOps, Cloud Native]
excerpt: Learn how to build and deploy Spring Boot applications using Docker and Kubernetes. This guide covers containerization, configuration, scaling, and cloud-native deployment strategies.
---
Deploying Spring Boot applications in modern cloud environments demands scalability, reliability, and automation. With **Docker** and **Kubernetes**, you can package, deploy, and manage your Java microservices with ease.

This guide covers how to:
- Containerize Spring Boot apps using Docker
- Create Kubernetes manifests
- Deploy applications in a cluster
- Scale and update services dynamically
- Optimize for production-grade operations

---

#### Why Docker and Kubernetes?

**Docker** allows you to bundle your app with all its dependencies in a lightweight, portable container. **Kubernetes** orchestrates these containers, handling:
- Service discovery
- Load balancing
- Auto-scaling
- Rolling deployments
- Self-healing

Together, they form a foundation for **cloud-native**, **microservice-based** architectures.

---

#### Step 1: Create a Spring Boot Application

Start with a basic Spring Boot app:

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello from Spring Boot on Kubernetes!";
    }
}
```

Build the app:

```bash
./mvnw clean package
```

---

#### Step 2: Create a Dockerfile

Here’s a sample Dockerfile for your Spring Boot JAR:

```dockerfile
FROM eclipse-temurin:17-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

Build the image:

```bash
docker build -t springboot-k8s-app .
```

Run locally to test:

```bash
docker run -p 8080:8080 springboot-k8s-app
```

---

#### Step 3: Push Docker Image to a Registry

To deploy on Kubernetes, the image must be hosted on a registry:

```bash
docker tag springboot-k8s-app your-dockerhub-username/springboot-k8s-app:v1
docker push your-dockerhub-username/springboot-k8s-app:v1
```

For private registries, configure access in Kubernetes using imagePullSecrets.

---

#### Step 4: Kubernetes Deployment YAML

Create `deployment.yaml`:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
name: springboot-app
spec:
replicas: 3
selector:
matchLabels:
app: springboot-app
template:
metadata:
labels:
app: springboot-app
spec:
containers:
- name: springboot-container
image: your-dockerhub-username/springboot-k8s-app:v1
ports:
- containerPort: 8080
```

---

#### Step 5: Kubernetes Service YAML

Expose the app using a service:

```yml
apiVersion: v1
kind: Service
metadata:
name: springboot-service
spec:
type: LoadBalancer
selector:
app: springboot-app
ports:
- protocol: TCP
port: 80
targetPort: 8080
```

Apply both files:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

#### Step 6: Access the Application

Use the external IP provided by your LoadBalancer service:

```bash
kubectl get service springboot-service
```

You’ll see:

```
springboot-service  LoadBalancer  xx.xx.xx.xx  80:xxxx/TCP
```

Navigate to `http://xx.xx.xx.xx/hello`.

---

#### Step 7: Enable Auto-scaling

Kubernetes supports **Horizontal Pod Autoscaling** (HPA) based on CPU usage:

```bash
kubectl autoscale deployment springboot-app --cpu-percent=50 --min=2 --max=10
```

Ensure your app exposes CPU metrics using Prometheus or `metrics-server`.

---

#### Step 8: Config Maps and Secrets

Store configs externally using:

```yml
apiVersion: v1
kind: ConfigMap
metadata:
name: app-config
data:
GREETING: "Welcome to the Kubernetes world!"
```

Inject via environment variable:

```yml
env:
- name: GREETING
  valueFrom:
  configMapKeyRef:
  name: app-config
  key: GREETING
  ```

---

#### Step 9: Zero-Downtime Deployments

Use rolling updates:

```bash
kubectl set image deployment/springboot-app springboot-container=your-dockerhub-username/springboot-k8s-app:v2
```

Kubernetes will:
- Create new pods
- Wait until ready
- Terminate old pods after transition

---

#### Step 10: Observability and Logging

Integrate with:
- **Prometheus + Grafana** for metrics
- **ELK or Loki stack** for logging
- **Jaeger** or **Zipkin** for tracing

Ensure you:
- Add readiness/liveness probes
- Export `/actuator/metrics` for observability
- Collect logs using sidecar or Fluentd agents

---

#### Conclusion

Using **Spring Boot with Docker and Kubernetes** unlocks a new level of scalability and automation for your applications. From containerization to deployment and monitoring, you can manage services efficiently across environments.

Whether you're running on local clusters with Minikube or in production on EKS, GKE, or AKS, these patterns help you build resilient, cloud-native Spring Boot systems that scale.
