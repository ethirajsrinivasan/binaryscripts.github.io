---
layout: post
title: Configuring HDFS for Multi Tenant Environments
subtitle: Set up HDFS to support secure, isolated, and efficient multi-tenant data platforms
categories: HDFS
tags: [HDFS, Hadoop, Multi-Tenancy, Access Control, Data Security, Quotas, Governance]
excerpt: Learn how to configure HDFS for secure and efficient multi-tenant usage. Explore techniques like directory permissions, user/group mapping, quotas, and access control integration using Ranger or Kerberos.
---
As organizations scale their data infrastructure, it’s common to support **multiple teams, departments, or clients** on a shared Hadoop cluster. This introduces the need for **multi-tenancy** — the ability to isolate and manage resources and data access securely and efficiently for each tenant.

This guide explores how to **configure HDFS for multi-tenant environments**, focusing on **security**, **resource governance**, **data access control**, and **storage quotas** to ensure safe, compliant, and cost-effective big data operations.

---

#### What is Multi-Tenancy in HDFS?

Multi-tenancy in HDFS allows multiple tenants (e.g., teams, projects, applications) to:
- Store and process data independently
- Maintain security and privacy boundaries
- Access only authorized data
- Track usage and apply quotas
- Co-exist without interference

Tenants share a common HDFS infrastructure but operate within **isolated logical environments**.

---

#### Directory Structure and Isolation

A clear **directory hierarchy** is essential:

```
/user/
├── finance/
├── engineering/
├── marketing/
├── shared/
```

Assign ownership and permissions:

```bash
hdfs dfs -mkdir /user/finance
hdfs dfs -chown finance:hadoop /user/finance
hdfs dfs -chmod 700 /user/finance
```

Use `700` for strict isolation, or `750` for read access by group members.

---

#### User and Group Management

Map HDFS users to **UNIX/Linux system users** or **Kerberos principals**.

Group users by tenant/team:

```bash
usermod -a -G finance alice
usermod -a -G engineering bob
```

In **Kerberos-enabled clusters**, principals follow this convention:

```
alice@CORP.COM → user: alice
bob@CORP.COM → user: bob
```

Use **Ranger** or **Sentry** for fine-grained role-based access control (RBAC).

---

#### Enforcing Directory Quotas

HDFS supports **space and file quotas** per directory:

- **Space quota**: Total bytes that can be stored
- **Namespace quota**: Number of files/directories

Set quotas:

```bash
hdfs dfsadmin -setSpaceQuota 500g /user/finance
hdfs dfsadmin -setQuota 1000000 /user/finance
```

Check quotas:

```bash
hdfs dfs -count -q /user/finance
```

This prevents tenants from consuming excessive disk space or metadata resources.

---

#### Enabling Access Control Lists (ACLs)

ACLs allow more flexible permissioning than UNIX-style permissions:

Enable ACL support:

```xml
<property>
<name>dfs.namenode.acls.enabled</name>
<value>true</value>
</property>
```

Apply ACLs:

```bash
hdfs dfs -setfacl -m user:alice:rwx /user/finance
hdfs dfs -setfacl -m group:auditors:rx /user/finance
```

List ACLs:

```bash
hdfs dfs -getfacl /user/finance
```

ACLs enable **shared access**, **auditor roles**, and **temporary permissions** for data consumers.

---

#### Auditing and Monitoring Access

For enterprise-grade multi-tenancy, enable **auditing** to monitor user activity:

- **Apache Ranger** provides audit trails of file access
- Use **HDFS audit logs** for raw audit data
- Integrate with **SIEM systems** (Splunk, ELK, etc.)

Enable HDFS audit logging:

```xml
<property>
<name>dfs.namenode.audit.log.enabled</name>
<value>true</value>
</property>
```

Track actions like read/write/delete per user and per path.

---

#### Resource Management with YARN Integration

While HDFS handles storage, compute resources must be governed using **YARN queues**.

Set up YARN **capacity scheduler**:

```xml
<property>
<name>yarn.scheduler.capacity.root.finance.capacity</name>
<value>30</value>
</property>
```

This reserves cluster CPU/memory for each tenant and prevents resource starvation.

Link HDFS usage to YARN queues for end-to-end resource accounting.

---

#### Best Practices

- Use **dedicated directories** per tenant with clear naming
- Apply **ownership and permissions** at directory level
- Enforce **quotas** to avoid resource abuse
- Enable **audit logging** and monitor with centralized tools
- Use **Ranger** or **Kerberos** for fine-grained access control
- Regularly review directory usage and cleanup stale data

---

#### Conclusion

Configuring HDFS for multi-tenancy ensures that multiple teams or clients can safely and efficiently share the same Hadoop infrastructure. By leveraging **directory-level isolation**, **quotas**, **ACLs**, and **audit logging**, you can build a secure, governed environment that supports scalable data operations.

With the right architecture and tools in place, HDFS becomes a robust foundation for modern, multi-tenant big data platforms.
