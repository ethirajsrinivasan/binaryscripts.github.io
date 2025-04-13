---
layout: post
title: Implementing Multi-Tenancy in Spring Boot Applications
subtitle: Architect Spring Boot applications for multi-tenant support using database and schema isolation strategies
categories: SpringBoot
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: [Java, Spring Boot, Multi-Tenancy, SaaS, Hibernate, Databases]
excerpt: Learn how to implement multi-tenancy in Spring Boot applications using schema-based, database-based, and discriminator-based approaches. Scale your SaaS solution while maintaining data isolation and performance.
---
As **Software-as-a-Service (SaaS)** platforms grow in popularity, building applications that serve multiple customers — or *tenants* — from a single codebase becomes critical. This is where **multi-tenancy** comes in.

Multi-tenancy is the architectural pattern that allows a single application to serve multiple clients while isolating their data. In this post, we’ll explore how to implement **multi-tenancy in Spring Boot** using different strategies like **schema-based**, **database-based**, and **discriminator-based** approaches, all while maintaining scalability and security.

---

#### What is Multi-Tenancy?

In multi-tenant applications:
- **Tenants** are logically separated entities (e.g., companies, users, accounts)
- Data must be **isolated** to prevent leakage
- Shared resources (code, app servers) are used for cost efficiency

**Types of multi-tenancy:**
1. **Database per tenant** – Physically isolated
2. **Schema per tenant** – Logically isolated
3. **Single schema (discriminator)** – Shared table, tenant ID field

---

#### When to Use Each Strategy

| Strategy         | Pros                             | Cons                                 |
|------------------|----------------------------------|--------------------------------------|
| Database-per-tenant | Strongest isolation, scalability | Harder to manage connections         |
| Schema-per-tenant   | Balance of isolation and control | Requires dynamic schema resolution   |
| Single schema       | Simplicity, easy to query       | Weak isolation, harder to scale      |

Your choice depends on:
- Data sensitivity
- Scalability needs
- Operational overhead

---

#### Setting Up a Spring Boot Multi-Tenant Project

Let’s build a schema-based multi-tenant solution using Hibernate.

Add dependencies:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
<groupId>org.hibernate</groupId>
<artifactId>hibernate-core</artifactId>
</dependency>
```

---

#### Tenant Resolver: Identify the Current Tenant

Implement a resolver that extracts the tenant ID from the request:

```java
@Component
public class TenantIdentifierResolver implements CurrentTenantIdentifierResolver {

    @Override
    public String resolveCurrentTenantIdentifier() {
        return TenantContext.getCurrentTenant(); // from ThreadLocal
    }

    @Override
    public boolean validateExistingCurrentSessions() {
        return true;
    }
}
```

Use a filter or interceptor to set the context:

```java
public class TenantFilter extends OncePerRequestFilter {
protected void doFilterInternal(...) {
String tenantId = request.getHeader("X-Tenant-ID");
TenantContext.setCurrentTenant(tenantId);
filterChain.doFilter(request, response);
TenantContext.clear();
}
}
```

---

#### Dynamic DataSource and Multi-Tenant Connection Provider

Configure a dynamic `DataSource` based on the tenant:

```java
@Component
public class SchemaBasedMultiTenantConnectionProvider implements MultiTenantConnectionProvider {

    private final DataSource dataSource;

    public SchemaBasedMultiTenantConnectionProvider(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Connection getConnection(String tenantIdentifier) throws SQLException {
        Connection connection = dataSource.getConnection();
        connection.createStatement().execute("SET search_path TO " + tenantIdentifier);
        return connection;
    }

    // Other methods omitted for brevity
}
```

---

#### Hibernate Configuration for Multi-Tenancy

Enable multi-tenancy in the Hibernate configuration:

```java
@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory(
DataSource dataSource,
MultiTenantConnectionProvider connectionProvider,
CurrentTenantIdentifierResolver tenantResolver) {

    Map<String, Object> properties = new HashMap<>();
    properties.put(Environment.MULTI_TENANT, MultiTenancyStrategy.SCHEMA);
    properties.put(Environment.MULTI_TENANT_CONNECTION_PROVIDER, connectionProvider);
    properties.put(Environment.MULTI_TENANT_IDENTIFIER_RESOLVER, tenantResolver);

    LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
    em.setPackagesToScan("com.example.app.domain");
    em.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
    em.setJpaPropertyMap(properties);
    em.setDataSource(dataSource);

    return em;
}
```

---

#### Securing Multi-Tenant Data Access

Use **Spring Security** and JWTs to enforce tenant isolation:

- Include `tenantId` in the token payload
- Validate that the authenticated user can access the tenant
- Apply security rules in service layers and repositories

Optionally add a `@TenantScoped` annotation to enforce tenant constraints at runtime.

---

#### Database per Tenant (Alternative)

For stronger isolation, configure one `DataSource` per tenant. Use a `RoutingDataSource` to delegate based on tenant ID:

```java
public class TenantRoutingDataSource extends AbstractRoutingDataSource {
@Override
protected Object determineCurrentLookupKey() {
return TenantContext.getCurrentTenant();
}
}
```

This allows separate databases, with different credentials and backup policies.

---

#### Testing and Monitoring Multi-Tenant Systems

**Test each tenant context** independently with integration tests.

Log the current tenant in each log line:

```yml
logging.pattern.level: "%5p [tenant=%X{tenant}]"
```

Use monitoring tools (e.g., Prometheus) to track tenant-specific metrics like:
- DB connection usage
- Request latency
- Error rates

---

#### Conclusion

Implementing multi-tenancy in Spring Boot requires careful consideration of your data model, security, and operational overhead. With the right strategy — whether schema-based or database-per-tenant — you can build scalable, secure SaaS platforms that serve many clients from a single, efficient codebase.

Spring Boot and Hibernate provide the hooks you need to build this architecture cleanly and effectively.
