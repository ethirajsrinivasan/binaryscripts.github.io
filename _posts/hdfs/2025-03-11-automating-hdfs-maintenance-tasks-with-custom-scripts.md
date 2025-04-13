---
layout: post
title: Automating HDFS Maintenance Tasks with Custom Scripts
subtitle: Streamline HDFS operations using shell, Python, and cron for consistent and efficient maintenance
categories: HDFS
tags: [HDFS, Hadoop, Automation, Shell Scripting, Cron, Maintenance, Big Data, DevOps]
excerpt: Learn how to automate key HDFS maintenance tasks using custom scripts. From data cleanup and archiving to health checks and quota monitoring, discover scripting techniques to keep your Hadoop cluster optimized.
---
Maintaining a healthy and efficient Hadoop cluster requires ongoing **HDFS maintenance** — tasks like cleaning up old files, checking disk usage, verifying replication, and managing quotas. While tools like Ambari or Cloudera Manager provide UIs, scripting these tasks provides **flexibility, automation, and reproducibility**.

In this blog, you'll learn how to **automate common HDFS maintenance tasks** using **custom shell and Python scripts**, integrate them into cron jobs, and ensure your storage layer runs smoothly with minimal manual intervention.

---

#### Why Automate HDFS Maintenance?

Manual HDFS management can be:
- Error-prone
- Time-consuming
- Inconsistent across teams

Automation helps you:
- Ensure regular cleanup and audits
- Avoid running out of disk space
- Detect and fix under/over-replicated blocks
- Enforce retention and quota policies
- Simplify routine administration

---

#### Task 1: Cleaning Up Old Files

Remove files older than a set number of days:

```bash
#!/bin/bash
TARGET_DIR="/data/logs/"
DAYS=30

hdfs dfs -find $TARGET_DIR -type f -mtime +$DAYS -delete
```

Schedule with cron:

```bash
0 3 * * * /usr/local/bin/hdfs-cleanup.sh >> /var/log/hdfs-cleanup.log 2>&1
```

---

#### Task 2: Checking Disk Usage and Generating Alerts

Monitor space usage on HDFS:

```bash
#!/bin/bash
hdfs dfs -du -h /data | sort -hr | head -n 10 > /tmp/top-hdfs-usage.txt
mail -s "Top HDFS Usage Report" admin@example.com < /tmp/top-hdfs-usage.txt
```

Use this to identify heavy users or runaway directories.

---

#### Task 3: Verifying Block Replication Health

Check for under-replicated or corrupt blocks:

```bash
#!/bin/bash
hdfs fsck / -blocks -locations -racks > /tmp/hdfs-fsck.log
grep -A5 'Under replicated' /tmp/hdfs-fsck.log
```

You can also integrate with alerting tools like Nagios, Zabbix, or Prometheus exporters.

---

#### Task 4: Monitoring and Enforcing Quotas

To detect quota breaches:

```bash
#!/bin/bash
hdfs dfs -count -q -h /user/* | awk '$1 != "-" && ($1 > $4 || $2 > $5)' > /tmp/quota-violations.txt

if [ -s /tmp/quota-violations.txt ]; then
mail -s "HDFS Quota Exceeded" admin@example.com < /tmp/quota-violations.txt
fi
```

This helps you proactively manage storage limits for teams or users.

---

#### Task 5: Automating Archival to Cold Storage

Move cold data to a separate directory or HDFS storage policy:

```bash
#!/bin/bash
SOURCE="/data/active"
DEST="/data/archive"
DAYS=60

hdfs dfs -find $SOURCE -type f -mtime +$DAYS | while read file; do
filename=$(basename "$file")
hdfs dfs -mv "$file" "$DEST/$filename"
done
```

Apply `COLD` storage policy:

```bash
hdfs storagepolicies -setStoragePolicy -path /data/archive -policy COLD
```

---

#### Task 6: Snapshot Creation for Compliance

Daily snapshot automation for backup:

```bash
#!/bin/bash
DIR="/user/secure"
DATE=$(date +%Y%m%d)
SNAPSHOT_NAME="snapshot_$DATE"

hdfs dfs -createSnapshot $DIR $SNAPSHOT_NAME
```

Cleanup old snapshots:

```bash
hdfs dfs -deleteSnapshot $DIR snapshot_$(date --date='30 days ago' +%Y%m%d)
```

---

#### Task 7: Python-Based Script for Reporting

A Python script for disk usage summary:

```python
import subprocess

output = subprocess.check_output(["hdfs", "dfs", "-du", "-h", "/data"])
lines = output.decode().splitlines()
for line in lines:
size, _, path = line.split(maxsplit=2)
print(f"{path}: {size}")
```

Schedule this with Airflow or cron for weekly dashboards.

---

#### Best Practices for HDFS Automation

- Use logging for every script (`>> logfile.log 2>&1`)
- Add alerts for failures (email, Slack, webhooks)
- Validate paths and quotas before deletion or moves
- Store scripts in version control (Git)
- Use a standard location for all jobs (`/usr/local/bin/hdfs-jobs/`)
- Use service accounts with minimal required permissions

---

#### Conclusion

Automating HDFS maintenance tasks using custom scripts helps ensure **operational consistency**, **storage efficiency**, and **proactive issue detection** in big data environments. Whether you're a Hadoop admin or DevOps engineer, scripting these tasks saves time, reduces human error, and strengthens your cluster's reliability.

Start small — clean up old logs, check quotas, run fsck — and build toward a fully automated, self-healing HDFS infrastructure.
