---
layout: post
title: Integrating Hudi with Kubernetes for Scalable Deployments
subtitle: Learn how to deploy Apache Hudi on Kubernetes to enable scalable, cloud-native data lake operations
categories: Hudi
tags: [Hudi, Kubernetes, Big Data, Lakehouse, Spark, Helm, Cloud Native, Scalable ETL]
excerpt: Explore how to run Apache Hudi on Kubernetes using Spark-on-K8s for scalable, resilient, and cloud-native data lake processing. Learn about Helm, containerization, resource tuning, and best practices.
---
Apache Hudi is a powerful framework for building data lakehouses with **upsert**, **incremental processing**, and **time-travel capabilities**. While Hudi traditionally runs on YARN or Spark Standalone, running it on **Kubernetes** offers significant advantages in terms of **scalability**, **cloud-native architecture**, and **resource efficiency**.

In this blog, we explore how to deploy and manage **Hudi workloads on Kubernetes**, leveraging **Spark on K8s**, **Helm charts**, and best practices to support real-time and batch data lake processing.

---

#### Why Run Hudi on Kubernetes?

| Benefit               | Description                                               |
|------------------------|-----------------------------------------------------------|
| Scalability           | Easily scale pods and Spark executors on demand           |
| Portability           | Run on any cloud or on-prem K8s cluster                   |
| Cost Efficiency       | Use autoscaling and spot instances                        |
| Isolation             | Containerized apps prevent resource contention            |
| Cloud-native Ops      | Seamless CI/CD, monitoring, and rollbacks                 |

Running Hudi on K8s aligns with **modern infrastructure standards**, especially for hybrid and multi-cloud architectures.

---

#### Architecture Overview

To run Hudi on Kubernetes:

- **Apache Spark 3.x** with Kubernetes support is required
- Hudi jobs are submitted using **Spark-on-Kubernetes mode**
- Data is stored in **S3, GCS, or HDFS**
- Optionally use **Hive Metastore**, **AWS Glue**, or **Data Catalog**
- Deploy using **kubectl**, **Helm**, or **SparkOperator**

```
[Dockerized Spark App] → [K8s Cluster] → [S3/HDFS]  
↘                ↘  
[Hive Metastore]    [Glue Catalog]  
```

---

#### Prerequisites

- Kubernetes Cluster (EKS, GKE, AKS, or local minikube)
- Spark 3.1+ built with K8s support and Hudi compatibility
- Hudi utilities bundle JAR
- Docker or Podman to build custom Spark images
- Access to object store (S3, GCS, etc.)

---

#### Step 1: Build a Hudi-Compatible Spark Docker Image

Create a Dockerfile:

```Dockerfile
FROM gcr.io/spark-operator/spark:v3.3.0

ADD hudi-spark3.3-bundle_2.12-0.14.0.jar /opt/spark/jars/
```

Then build and push:

```bash
docker build -t myrepo/spark-hudi:latest .
docker push myrepo/spark-hudi:latest
```

---

#### Step 2: Submit Hudi Jobs on Kubernetes

Use Spark submit with `--master k8s` and point to your image:

```bash
spark-submit \
--master k8s://https://<k8s-api-server> \
--deploy-mode cluster \
--name hudi-job \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
--conf spark.executor.instances=5 \
--conf spark.kubernetes.container.image=myrepo/spark-hudi:latest \
--conf spark.kubernetes.namespace=default \
--conf spark.hadoop.fs.s3a.access.key=XXX \
--conf spark.hadoop.fs.s3a.secret.key=XXX \
local:///opt/spark/jars/hudi-utilities-bundle.jar \
--table-type COPY_ON_WRITE \
--target-base-path s3a://my-datalake/hudi/orders \
--target-table orders \
--props /etc/hudi/orders.properties
```

---

#### Step 3: Deploy via Helm (Optional)

Use Helm for templated deployment:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install spark bitnami/spark \
--set image.repository=myrepo/spark-hudi \
--set image.tag=latest \
--set worker.replicaCount=5
```

This simplifies provisioning Spark/Hudi workloads with autoscaling and monitoring.

---

#### Configurations for Performance

- **Shuffle tuning**:
  ```
  spark.sql.shuffle.partitions=200
  ```

- **Resource allocation**:
  ```
  spark.executor.memory=4g  
  spark.driver.memory=2g  
  spark.executor.cores=2  
  ```

- **Filesystem compatibility**:
  Ensure correct S3A/GCS/HDFS credentials in `spark.hadoop.*` properties.

---

#### Best Practices

- **Use SparkOperator** for managed job orchestration
- Enable **Hudi metadata table** for file pruning and faster planning
- Store checkpoints in object storage or persistent volume
- Use **initContainers** to load configuration dynamically
- Monitor jobs using **Prometheus + Grafana**, or Spark UI on K8s Dashboard
- Rotate logs and retain job history with volume mounts

---

#### Security Considerations

- Store credentials in **Kubernetes secrets**
- Enable **TLS encryption** between Spark driver and executor pods
- Use **RBAC policies** to limit access to Spark/K8s resources
- Integrate with **OIDC or IRSA (for AWS)** for fine-grained access

---

#### Conclusion

Deploying Apache Hudi on Kubernetes unlocks scalable, elastic, and cloud-native data processing workflows. With containerized Spark jobs, seamless orchestration, and resource isolation, organizations can modernize their ETL pipelines and support real-time lakehouse operations at scale.

Whether you're building a cloud-native analytics platform or transitioning from legacy YARN infrastructure, integrating Hudi with Kubernetes is a future-proof step toward **efficient and modular big data processing**.
