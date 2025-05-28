---
layout: post
title: Setting Up Continuous Integration and Deployment in Kubernetes with Jenkins and GitLab
subtitle: A detailed guide to implementing robust CI/CD pipelines on Kubernetes using Jenkins and GitLab
categories: Kubernetes
tags: [Kubernetes, CI/CD, Jenkins, GitLab, DevOps, Automation, Continuous Integration, Continuous Deployment, Pipelines, Containers]
excerpt: Learn how to build and automate CI/CD pipelines in Kubernetes environments leveraging Jenkins and GitLab for seamless integration and deployment workflows.
---
In modern software development, **Continuous Integration and Continuous Deployment (CI/CD)** pipelines are essential to deliver code rapidly, reliably, and at scale. Kubernetes, with its container orchestration capabilities, complements CI/CD workflows by providing a scalable, consistent runtime environment.

This post explores how to set up **CI/CD pipelines in Kubernetes** using two popular tools: **Jenkins** and **GitLab CI/CD**. We will cover architecture considerations, pipeline configuration, and best practices to optimize your deployment workflows.

---

#### Why CI/CD on Kubernetes?

Kubernetes offers:

- Containerized, reproducible environments for builds and deployments
- Automated scaling of build agents and runners
- Isolation and resource management for CI jobs
- Integration with cloud-native tools and secret management

Combining Kubernetes with Jenkins or GitLab enhances automation and accelerates delivery cycles.

---

#### Jenkins on Kubernetes: Architecture Overview

Jenkins supports Kubernetes through the **Kubernetes Plugin**, which enables dynamic provisioning of build agents (pods) on demand.

##### Core components:

- **Jenkins Master:** Coordinates jobs, pipelines, and stores configuration
- **Kubernetes Cluster:** Hosts ephemeral Jenkins agents as pods
- **Jenkins Agents:** Spin up dynamically using pod templates, execute CI tasks, then terminate

This setup allows scaling build capacity horizontally without managing static agents.

---

#### Setting Up Jenkins CI/CD Pipelines on Kubernetes

##### Step 1: Deploy Jenkins in Kubernetes

Deploy Jenkins using the official Helm chart:

```bash
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
helm install jenkins jenkinsci/jenkins --namespace jenkins --create-namespace
```

##### Step 2: Configure Kubernetes Plugin

- Install the **Kubernetes Plugin** on Jenkins
- Configure Kubernetes cloud settings to connect to your cluster API
- Define **Pod Templates** for different build environments (e.g., Maven, Node.js, Docker)

##### Step 3: Create Declarative Pipelines

Example Jenkinsfile using a Kubernetes agent:

```groovy
pipeline {
  agent {
    kubernetes {
      label 'jenkins-agent'
      yaml """
      apiVersion: v1
      kind: Pod
      spec:
        containers:
        - name: maven
          image: maven:3.8.1-jdk-11
          command:
          - cat
          tty: true
      """
    }
  }
  stages {
    stage('Build') {
      steps {
        container('maven') {
          sh 'mvn clean package'
        }
      }
    }
    stage('Deploy') {
      steps {
        sh 'kubectl apply -f deployment.yaml'
      }
    }
  }
}
```

---

#### GitLab CI/CD with Kubernetes Integration

GitLab CI/CD natively supports Kubernetes via its **Kubernetes Integration**, allowing runners to schedule jobs directly on the cluster.

##### Step 1: Connect GitLab to Kubernetes Cluster

- In GitLab, navigate to your project’s **Operations > Kubernetes** section
- Add your cluster with API endpoint, CA certificate, and token

##### Step 2: Register GitLab Runner

Register a **GitLab Runner** configured to use Kubernetes executor:

```bash
gitlab-runner register \
  --executor kubernetes \
  --kubernetes-namespace gitlab \
  --description "Kubernetes Runner"
```

##### Step 3: Define `.gitlab-ci.yml`

Example pipeline file deploying to Kubernetes:

```yaml
stages:
  - build
  - deploy

build-job:
  image: maven:3.8-jdk-11
  stage: build
  script:
    - mvn clean package

deploy-job:
  image: bitnami/kubectl
  stage: deploy
  script:
    - kubectl apply -f deployment.yaml
  only:
    - main
```

---

#### Best Practices for CI/CD in Kubernetes

- **Use namespaces** to isolate environments (dev, staging, production)
- **Secure secrets** with Kubernetes Secrets or HashiCorp Vault
- **Automate rollback** strategies in deployment pipelines
- Implement **automated testing** at build stages
- Use **Helm charts** or **Kustomize** for managing Kubernetes manifests
- Monitor pipeline performance and failures with tools like Prometheus and Grafana

---

#### Conclusion

Setting up CI/CD pipelines in Kubernetes using Jenkins and GitLab unlocks automated, scalable, and reliable software delivery. Both tools offer flexible integrations tailored to Kubernetes environments, enabling teams to accelerate development cycles with confidence.

Implementing best practices and leveraging Kubernetes-native capabilities ensures your pipelines remain resilient, secure, and easy to maintain.

