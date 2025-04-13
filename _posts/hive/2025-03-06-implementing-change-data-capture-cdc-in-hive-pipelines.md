---
layout: post
title: Implementing Change Data Capture CDC in Hive Pipelines
subtitle: Track and manage incremental changes in Hive data lakes using CDC strategies and tools
categories: Hive
tags: [Hive, CDC, Change Data Capture, Big Data, ETL, Data Lake, Apache Hudi, Incremental Processing]
excerpt: Learn how to implement Change Data Capture (CDC) in Hive pipelines to track inserts, updates, and deletes. Explore strategies using SCD, audit columns, Hudi, and Hive ACID for efficient incremental processing.
---
As data lakes grow, so does the need for efficient **incremental processing**. Full table scans are wasteful when you're only interested in the *new* or *changed* data. That's where **Change Data Capture (CDC)** comes in — enabling Hive to detect inserts, updates, and deletes for downstream processing.

This blog explains **how to implement CDC in Hive pipelines**, using classic techniques like Slowly Changing Dimensions (SCD), audit columns, and modern solutions like **Apache Hudi**, **Delta Lake**, and **Hive ACID tables**.

---

#### What is Change Data Capture?

**Change Data Capture (CDC)** is a design pattern used to identify and process only the *changed* data (inserted, updated, or deleted) in a source system.

CDC is essential for:
- Real-time data ingestion
- Data warehouse synchronization
- ETL pipeline efficiency
- Accurate reporting and dashboards

In Hive, CDC isn't native to traditional external tables, but it can be implemented with **metadata, file management**, and transactional storage formats.

---

#### Strategy 1: Audit Columns with Timestamps

Add audit columns such as `last_updated_ts` to track changes:

```sql
CREATE TABLE customers (
customer_id STRING,
name STRING,
email STRING,
last_updated_ts TIMESTAMP
)
STORED AS PARQUET;
```

Your ETL pipeline can filter records based on the last checkpoint:

```sql
SELECT * FROM customers
WHERE last_updated_ts > '${last_run_timestamp}';
```

This method is simple and widely used when source systems support timestamp-based deltas.

---

#### Strategy 2: Using Operation Type Columns

Add an operation flag (`I`, `U`, `D`) to capture changes in ingestion pipelines:

```sql
CREATE TABLE cdc_raw (
id STRING,
data STRING,
op_type STRING,  -- 'I', 'U', 'D'
ts TIMESTAMP
)
STORED AS PARQUET;
```

Then, apply the logic during merge operations:

```sql
-- pseudo code
IF op_type = 'I' THEN insert
IF op_type = 'U' THEN update
IF op_type = 'D' THEN delete
```

You can implement this logic using **MERGE INTO** with Apache Hudi or Hive ACID (covered below).

---

#### Strategy 3: Apache Hudi for CDC in Hive

**Apache Hudi** adds transactional and CDC capabilities directly into Hive-compatible data lakes.

Create a Hudi table in Hive:

```sql
CREATE TABLE hudi_customers (
customer_id STRING,
name STRING,
email STRING,
ts TIMESTAMP
)
USING hudi
TBLPROPERTIES (
'hoodie.datasource.write.recordkey.field' = 'customer_id',
'hoodie.datasource.write.precombine.field' = 'ts',
'hoodie.table.type' = 'MERGE_ON_READ'
);
```

Hudi supports:
- **Incremental reads**
- **Merge-on-read vs. copy-on-write**
- **Efficient upserts and deletes**

To query only changes:

```sql
SELECT * FROM hudi_customers
WHERE `_hoodie_commit_time` > '${last_commit_time}';
```

Hudi integrates with Hive via **Hive sync**, letting you run CDC queries directly from HiveQL.

---

#### Strategy 4: Hive ACID Transactions

For pure Hive-based CDC, use **ACID transactional tables** introduced in Hive 3+:

```sql
CREATE TABLE customer_acid (
id STRING,
name STRING,
email STRING
)
CLUSTERED BY (id) INTO 3 BUCKETS
STORED AS ORC
TBLPROPERTIES ('transactional'='true');
```

ACID tables support **INSERT**, **UPDATE**, and **DELETE**, enabling fine-grained row-level changes.

Example update:

```sql
UPDATE customer_acid SET email = 'new@email.com' WHERE id = '123';
```

To capture changes:
- Use **ORC base and delta files**
- Track using **compaction and write IDs**

Drawback: Requires Hive LLAP or Tez, and is resource intensive for very large datasets.

---

#### Strategy 5: External CDC with Kafka + Flink + Hive Sink

If you're ingesting data from operational databases (like MySQL/Postgres), use:
- **Debezium** to capture CDC from source
- **Apache Kafka** to transport changes
- **Apache Flink or Spark** to transform data
- **Hive Sink Connector** to write into Hive-compatible formats

This creates an end-to-end CDC pipeline with near real-time updates flowing into Hive tables.

---

#### Choosing the Right CDC Approach

| Approach              | Pros                            | Cons                              |
|-----------------------|----------------------------------|-----------------------------------|
| Audit Columns         | Simple, easy to query            | Cannot detect deletes             |
| Op Type Flags         | Supports full DML                | Needs logic in ETL                |
| Apache Hudi           | Scalable, supports incremental   | Requires setup, resource overhead |
| Hive ACID Tables      | Native updates/deletes           | Slower, complex setup             |
| Kafka + Flink         | Real-time, stream processing     | High complexity and latency       |

Choose the right CDC strategy based on:
- Data volume
- Latency requirements
- Tooling and team expertise
- Compatibility with your data lake platform

---

#### Best Practices

- Use **Hudi** or **Delta Lake** for lake-native CDC
- Store CDC metadata (source system, timestamp, operation type)
- Test incremental logic against edge cases (nulls, reordering)
- Avoid excessive updates — prefer immutable design when possible
- Automate CDC watermarking to track last processed state

---

#### Conclusion

Implementing **Change Data Capture (CDC)** in Hive enables you to build efficient, reliable, and incremental ETL pipelines. Whether you're using Hive ACID, Apache Hudi, or external CDC tools, the right strategy will reduce compute costs and improve data freshness.

By mastering CDC in Hive, your data workflows become more real-time, scalable, and analytics-ready.
