---
layout: post
title: Automating HDFS Data Archival with Apache Oozie
subtitle: Learn how to use Apache Oozie to automate and schedule HDFS data archival workflows in Hadoop
categories: HDFS
tags: [HDFS, Oozie, Hadoop, Data Archival, Automation, Workflow, Big Data]
excerpt: Discover how to use Apache Oozie to automate HDFS data archival in Hadoop. Learn how to schedule, move, and manage old data efficiently with coordinated workflows.
---
As data accumulates in Hadoop clusters, managing storage becomes a critical task. Frequently accessed data should stay on fast storage, while **historical or infrequently used data** can be **archived to cost-efficient locations**.

Automating this process with **Apache Oozie**, the workflow scheduler for Hadoop, allows organizations to **systematically move data** within HDFS — reducing costs and keeping clusters optimized for performance.

In this post, we'll walk through how to use **Apache Oozie to automate HDFS data archival**, define workflows, schedule them with coordinators, and apply best practices for production-grade automation.

---

#### Why Archive Data in HDFS?

Data archival helps:
- Free up HDFS space for active datasets
- Improve NameNode performance by reducing file count
- Lower storage costs by moving data to **low-tier or compressed directories**
- Comply with data retention policies

Common use cases:
- Archiving old logs
- Archiving time-partitioned data (e.g., daily, monthly)
- Preparing data for long-term storage (e.g., S3 or cold HDFS)

---

#### What is Apache Oozie?

**Apache Oozie** is a workflow scheduler system for managing Hadoop jobs. It supports:

- **Workflow jobs** (sequences of actions)
- **Coordinator jobs** (triggered by time/frequency/data conditions)
- **Bundle jobs** (grouped coordinators)

You can run HDFS commands, Pig scripts, Hive queries, and custom Java actions using Oozie.

---

#### Example Archival Workflow Design

We’ll design a simple workflow that:
1. Identifies directories older than a threshold (e.g., 30 days)
2. Moves them to an `/archive/` location
3. Optionally compresses the archived files

---

#### Step 1: Write a Shell Script for Archival Logic

Save this as `archive-data.sh` in HDFS or shared NFS:

```bash
#!/bin/bash
SOURCE_DIR=$1
ARCHIVE_DIR=$2
DAYS=$3

hdfs dfs -ls ${SOURCE_DIR} | awk '{print $8}' | while read line
do
file_date=$(basename "$line")
threshold=$(date -d "-${DAYS} days" +%Y-%m-%d)
if [[ "$file_date" < "$threshold" ]]; then
hdfs dfs -mv "$line" "${ARCHIVE_DIR}/$file_date"
fi
done
```

---

#### Step 2: Create a Workflow XML (`workflow.xml`)

```xml
<workflow-app name="hdfs-archival-wf" xmlns="uri:oozie:workflow:0.5">
<start to="archive-data"/>

  <action name="archive-data">
    <shell xmlns="uri:oozie:shell-action:0.2">
      <job-tracker>${jobTracker}</job-tracker>
      <name-node>${nameNode}</name-node>
      <exec>archive-data.sh</exec>
      <argument>${sourceDir}</argument>
      <argument>${archiveDir}</argument>
      <argument>${retentionDays}</argument>
      <file>archive-data.sh#archive-data.sh</file>
    </shell>
    <ok to="end"/>
    <error to="fail"/>
  </action>

  <kill name="fail">
    <message>Archival failed - check logs</message>
  </kill>

  <end name="end"/>
</workflow-app>
```

---

#### Step 3: Define Coordinator XML (`coordinator.xml`)

This schedules the workflow daily:

```xml
<coordinator-app name="hdfs-archival-coord" frequency="1" start="${coordStart}" end="${coordEnd}" timezone="UTC" xmlns="uri:oozie:coordinator:0.4">
<action>
<workflow>
<app-path>${workflowAppUri}</app-path>
<configuration>
<property>
<name>sourceDir</name>
<value>/data/logs</value>
</property>
<property>
<name>archiveDir</name>
<value>/archive/logs</value>
</property>
<property>
<name>retentionDays</name>
<value>30</value>
</property>
</configuration>
</workflow>
</action>
</coordinator-app>
```

---

#### Step 4: Submit the Coordinator Job

1. Package the workflow:

```bash
oozie job -oozie http://oozie-server:11000/oozie -config job.properties -run
```

2. Sample `job.properties`:

```
nameNode=hdfs://namenode:8020
jobTracker=jobtracker:8021
workflowAppUri=/user/oozie/hdfs-archival
coordStart=2024-11-01T00:00Z
coordEnd=2024-12-31T00:00Z
oozie.use.system.libpath=true
oozie.libpath=/user/oozie/share/lib
```

---

#### Best Practices

- **Parameterize directories and retention periods** for reusability
- Use **shell scripts or Spark actions** for custom archival logic
- **Test workflows manually** before scheduling
- Integrate with **Apache Ranger** for secure access control
- Monitor via **Oozie UI** or **Apache Ambari**

---

#### Conclusion

Automating HDFS data archival with Apache Oozie simplifies the management of growing data volumes. By scheduling archival workflows, you ensure that your Hadoop clusters stay lean, performant, and compliant with data retention policies.

With Oozie's flexibility and HDFS's scalability, you can build robust data lifecycle management workflows tailored to your enterprise's storage needs.
