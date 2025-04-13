---
layout: post
title: Understanding Hudi Commit Timeline and Versioning
subtitle: Explore how Apache Hudi manages commit timelines to enable data versioning, rollback, and time-travel
categories: Hudi
tags: [Hudi, Apache Hudi, Commit Timeline, Versioning, Time Travel, Lakehouse, Big Data]
excerpt: Dive deep into Apache Hudi's commit timeline and learn how it powers version control, incremental queries, rollback, and time-travel on data lakes. Master the timeline file structure and lifecycle states.
---
Apache Hudi brings **data versioning**, **incremental processing**, and **time-travel queries** to your data lake. These powerful features are made possible by Hudi’s **commit timeline**, a metadata structure that tracks the **history of operations** performed on a dataset.

In this guide, we’ll explore the **Hudi commit timeline**, how it manages file versions, what different commit states mean, and how to leverage it for **debugging, rollback, auditing**, and **incremental ETL pipelines**.

---

#### What is the Hudi Commit Timeline?

The **commit timeline** is a sequence of **instant files** stored in the `.hoodie` metadata directory of a Hudi table. Each instant file represents a specific action taken on the dataset, such as:
- Inserts
- Upserts
- Deletes
- Compactions
- Cleanups
- Rollbacks

Each instant is named using a **timestamp**, which acts as a unique version identifier.

Example timeline files:

```
.hoodie/
├── 20240412103015.commit
├── 20240412104522.commit
├── 20240412120033.inflight
├── 20240412130000.clean
├── 20240412140000.rollback
```

---

#### Instant Types and Lifecycle

Every action creates an **instant**, and it can be in one of several **states**:

| State       | Description                                         |
|-------------|-----------------------------------------------------|
| **Requested** | Operation is scheduled but not started            |
| **Inflight**  | Operation is currently running                    |
| **Completed** | Operation finished successfully                   |
| **Rollback**  | Reverts a previously failed or partial operation  |

Common types of instants:

- `.commit`, `.compaction`, `.delta_commit` → **successful writes**
- `.inflight` → **ongoing write**
- `.rollback` → **reverted operations**
- `.clean`, `.savepoint`, `.replacecommit` → **maintenance events**

---

#### Commit Timeline for Copy-on-Write vs Merge-on-Read

| Table Type      | Instant Types Used            | Description                         |
|------------------|-------------------------------|-------------------------------------|
| Copy-on-Write    | `.commit`                     | Writes produce base files directly  |
| Merge-on-Read    | `.delta_commit`, `.compaction`| Log files + base files (with merge) |

Merge-on-Read stores changes in **log files** first (via delta commits), and periodically merges them into base files through **compaction**.

---

#### Use Cases for the Commit Timeline

1. **Data Versioning**

Each commit represents a consistent snapshot of the dataset. You can **replay or restore** any version using the commit timestamp.

```sql
SELECT * FROM hudi_table
WHERE _hoodie_commit_time = '20240412103015';
```

2. **Time-Travel Queries**

Retrieve data as of a specific time using **Hudi's built-in timestamp filtering**.

```sql
SELECT * FROM hudi_table
WHERE _hoodie_commit_time <= '20240412120000';
```

3. **Incremental ETL Pipelines**

Query only records changed since the last commit:

```python
df = spark.read.format("hudi") \
.option("hoodie.datasource.query.type", "incremental") \
.option("hoodie.datasource.read.begin.instanttime", "20240412104522") \
.load("s3://my-table")
```

4. **Rollback and Recovery**

Automatically or manually rollback failed writes:

```bash
hoodie-cli
> rollback --instant 20240412120033
```

5. **Auditing and Debugging**

Check file histories, modified partitions, and commit logs for tracing changes.

---

#### Viewing the Timeline with Hudi CLI

Use the CLI to inspect commit history:

```bash
hoodie-cli
> connect --path s3://your-hudi-table
> show commits
> show commit --commit 20240412103015
> show fsview all
```

This shows:
- Commits and operations performed
- Affected partitions and files
- Metadata and statistics per commit

---

#### File Layout and Metadata

Each commit file contains:
- **commit metadata**: JSON with operation details
- **partition and file listings**
- **record count**, **write stats**, **errors**

Log file layout (MOR):

```
partition/
├── file1.parquet
├── .file1_1.log
├── .file1_2.log
├── .file1_3.log
```

These logs are merged during compaction.

---

#### Best Practices

- Use **incremental queries** for efficient CDC and streaming ETL
- Always monitor for **inflight** or **rollback** states — these may indicate failure
- Enable **savepoints** before large changes for safe recovery
- Compact **MOR tables** regularly to reduce delta file size
- Track timeline size and consider archiving old commits to avoid metadata bloat

---

#### Conclusion

Apache Hudi’s **commit timeline** is the backbone of its powerful lakehouse capabilities — from **data versioning** and **time-travel** to **incremental ingestion** and **rollback support**.

Understanding how the timeline works is essential for managing production-grade Hudi datasets with confidence. With proper use, it transforms your data lake into a fully **versioned, queryable, and auditable** platform — ready for real-time and historical analytics.
