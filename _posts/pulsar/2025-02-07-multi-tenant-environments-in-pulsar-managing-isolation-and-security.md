---
layout: post
title: Multi Tenant Environments in Pulsar Managing Isolation and Security
subtitle: Learn how to build secure and isolated multi-tenant messaging systems using Apache Pulsar
categories: Pulsar
tags: [Pulsar, Multi-Tenancy, Messaging, Security, Apache Pulsar, Streaming Architecture, RBAC, Isolation]
excerpt: Apache Pulsar natively supports multi-tenancy, enabling secure isolation between tenants. This post explores best practices for tenant management, authentication, authorization, and namespace configuration.
---
In today’s SaaS-driven world, **multi-tenancy** is a crucial capability for platforms that serve multiple customers, applications, or business units from a shared infrastructure. Apache Pulsar stands out from other messaging systems by offering **first-class support for multi-tenant environments**, allowing for secure and scalable isolation of data and workloads.

This post dives deep into how to implement and manage **multi-tenant messaging architectures in Pulsar**, with a focus on **security**, **isolation**, and **best practices**.

---

#### What is Multi-Tenancy in Pulsar?

Apache Pulsar is designed from the ground up with **multi-tenancy in mind**. It introduces hierarchical levels of **Tenants**, **Namespaces**, and **Topics** to structure and segregate workloads.

- **Tenant**: Logical grouping for customers, business units, or apps
- **Namespace**: Subdivision under a tenant for organizing topics and applying policies
- **Topic**: Actual messaging endpoint

This structure provides **fine-grained access control** and **logical isolation**, allowing multiple tenants to coexist without interference.

---

#### Core Building Blocks for Isolation

| Layer        | Purpose                                  | Example                         |
|--------------|------------------------------------------|----------------------------------|
| Tenant       | Represents a tenant or customer          | `tenantA`                        |
| Namespace    | Grouping within tenant for policies      | `tenantA/prod-namespace`         |
| Topic        | Stream endpoint                          | `persistent://tenantA/prod-namespace/orders` |

Each layer supports its own **ACLs, quotas, and security policies**.

---

#### Step 1: Create Tenants and Namespaces

Use the Pulsar Admin CLI or REST API:

```bash
# Create tenant
bin/pulsar-admin tenants create tenantA \
--allowed-clusters us-east-cluster \
--admin-roles tenantA-admin

# Create namespace
bin/pulsar-admin namespaces create tenantA/prod-namespace
```

Assign **cluster-level access** and **admin roles** at the tenant level.

---

#### Step 2: Configure Authentication and Authorization

Enable **authentication** in `broker.conf`:

```properties
authenticationEnabled=true
authenticationProviders=org.apache.pulsar.broker.authentication.AuthenticationProviderToken
authorizationEnabled=true
authorizationProvider=org.apache.pulsar.broker.authorization.PulsarAuthorizationProvider
```

Use **JWT tokens** or **TLS certificates** for identity:

```bash
# Assign role to namespace
bin/pulsar-admin namespaces grant-permission tenantA/prod-namespace \
--actions produce,consume \
--role tenantA-consumer
```

---

#### Step 3: Apply Resource Limits and Quotas

Namespaces can have **message and throughput quotas**:

```bash
# Limit to 100 MB/s publish rate
bin/pulsar-admin namespaces set-publish-rate tenantA/prod-namespace \
--msg-rate 1000 \
--byte-rate 104857600
```

Other controls include:
- **Subscription limits**
- **Retention policies**
- **Message TTL**
- **Offload policies**

This ensures tenants do not impact each other’s performance.

---

#### Step 4: Enable Audit Logging and Monitoring

Track access and usage per tenant:
- Enable **broker logs** to capture producer/consumer activity
- Use **Prometheus metrics** with Pulsar Exporter:
  - `pulsar_subscriptions_count{tenant="tenantA"}`
  - `pulsar_throughput_in{tenant="tenantA"}`

Visualize with **Grafana** dashboards per tenant/namespace.

---

#### Step 5: Isolate Tenants at the Broker or Cluster Level

For strict isolation:
- Use **dedicated clusters or brokers per tenant**
- Separate **network namespaces or Kubernetes pods**
- Apply **network policies and quotas**

In Kubernetes, deploy **per-tenant Pulsar clusters** via **Helm or Pulsar Operator** for stronger boundaries.

---

#### Security Best Practices

✅ Use **token-based authentication** with expiration  
✅ Implement **RBAC policies** using roles and scopes  
✅ Limit access to **namespaces**, not just topics  
✅ Enable **TLS encryption** for all data in transit  
✅ Monitor **per-tenant metrics and logs**  
✅ Separate **staging and production namespaces** per tenant

---

#### Common Use Cases

- **SaaS platforms** offering messaging as a service
- **Multi-region applications** with isolated environments
- **Business units** using shared Pulsar infrastructure
- **Dev vs Prod isolation** for microservice event streams

---

#### Conclusion

Apache Pulsar provides **native support for multi-tenancy**, making it an excellent choice for platforms that require **tenant isolation**, **security**, and **scalable governance**. With namespaces, RBAC, quotas, and flexible deployment options, Pulsar enables you to confidently serve multiple teams or customers without sacrificing control or performance.

By combining thoughtful design and proper policy enforcement, you can build a **robust, secure, multi
