---
layout: post
title: Managing Container Logs with ELK Stack Elasticsearch Logstash and Kibana
subtitle: Learn how to centralize and analyze container logs using Elasticsearch, Logstash, and Kibana (ELK Stack)
categories: Docker
tags: [ELK, Elasticsearch, Logstash, Kibana, Logging, Containers, Docker, Observability]
excerpt: Discover how to collect, process, and visualize container logs using the ELK Stack. This guide explains best practices and setup steps for centralized logging in containerized environments.
---
In modern DevOps environments, applications often run in **containers** using platforms like Docker and Kubernetes. While containers provide agility and scalability, they introduce challenges for logging — such as **log sprawl**, **short lifespans**, and **ephemeral environments**.

The **ELK Stack** — **Elasticsearch**, **Logstash**, and **Kibana** — offers a powerful solution to **centralize**, **process**, and **visualize** container logs in real time.

In this post, we’ll walk through how to set up and use the ELK stack for **managing container logs**, with an emphasis on **Docker environments**.

---

#### Why Use ELK for Container Logs?

The ELK Stack provides:

- **Elasticsearch**: Scalable search and analytics engine
- **Logstash**: Pipeline for collecting and transforming logs
- **Kibana**: Visualization and dashboarding interface

Together, they offer:

- Centralized log aggregation
- Real-time search and analysis
- Customizable dashboards and alerting
- Scalable storage and indexing

---

#### Architecture Overview

```
[Docker Containers]  
↓ stdout/stderr  
[Filebeat / Logstash]  
↓  
[Elasticsearch]  
↓  
[Kibana Dashboard]
```

You can use **Filebeat** or **Logstash** as the log collector from Docker containers, then send parsed logs to **Elasticsearch**, and visualize them in **Kibana**.

---

#### Step 1: Configure Docker Logging

By default, Docker logs go to **json-file** driver. Check it using:

```bash
docker inspect <container_id> | grep LogPath
```

To make logs accessible:

1. Use the default json driver (for Filebeat or Logstash to read).
2. Or configure log driver as syslog/gelf for direct shipping.

---

#### Step 2: Install and Configure Logstash

Logstash collects and parses log files.

Sample Logstash pipeline (logstash.conf):

```
input {
file {
path => "/var/lib/docker/containers/*/*.log"
type => "docker"
start_position => "beginning"
sincedb_path => "/dev/null"
codec => "json"
}
}

filter {
mutate {
add_field => { "container" => "%{[docker][container][name]}" }
}
}

output {
elasticsearch {
hosts => ["http://localhost:9200"]
index => "docker-logs-%{+YYYY.MM.dd}"
}
}
```

Run Logstash:

```bash
bin/logstash -f logstash.conf
```

---

#### Step 3: Set Up Elasticsearch

Install Elasticsearch and run:

```bash
bin/elasticsearch
```

Make sure the port `9200` is reachable and health check shows green/yellow.

Check logs:

```bash
curl -X GET "localhost:9200/_cat/indices?v"
```

---

#### Step 4: Install and Access Kibana

Install and launch Kibana:

```bash
bin/kibana
```

Access at: [http://localhost:5601](http://localhost:5601)

- Create an index pattern: `docker-logs-*`
- Visualize logs using:
  - Time series charts
  - Filters (container ID, log level, message)
  - Saved searches and dashboards

---

#### Optional: Using Filebeat Instead of Logstash

**Filebeat** is lightweight and suitable for edge logging.

Sample filebeat.yml:

```yaml
filebeat.inputs:
- type: container
  paths:
  - /var/lib/docker/containers/*/*.log
    processors:
  - add_docker_metadata: ~

output.elasticsearch:
hosts: ["localhost:9200"]
```

Start Filebeat:

```bash
./filebeat -e -c filebeat.yml
```

---

#### Security and Scalability Tips

- Enable **TLS** on Elasticsearch and Kibana
- Use **X-Pack** for authentication and RBAC
- Configure **index lifecycle management (ILM)** to manage disk usage
- Scale Elasticsearch with **shards**, **replicas**, and **hot-warm architecture**
- Apply **log rotation** and **volume mounts** to persist container logs

---

#### Use Cases

- **Microservices troubleshooting** with filters by container/service
- **Anomaly detection** via threshold alerts
- **DevOps observability dashboards**
- **Security audit logs** from containers
- **Resource optimization** by tracking log volume per container

---

#### Conclusion

The ELK Stack is a powerful toolset for **container log management**, enabling teams to observe, analyze, and act on data in real time. By integrating Docker with Logstash, Elasticsearch, and Kibana (or Filebeat), you can build a **centralized logging solution** that enhances visibility and accelerates issue resolution in complex environments.

Whether you're running Docker, Kubernetes, or hybrid environments, **centralized logging is key to observability**, and ELK gives you the tools to make it happen.
