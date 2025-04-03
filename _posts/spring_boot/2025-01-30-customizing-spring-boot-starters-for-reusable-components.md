---
layout: post
title: Implementing Multi-Tenancy in Spring Boot Applications
subtitle: Build scalable, tenant-aware applications with Spring Boot using schema and database separation strategies
categories: SpringBoot
tags: [Spring Boot, Java, Multi-Tenancy, Hibernate, JPA, SaaS, Architecture]
excerpt: Learn how to implement multi-tenancy in Spring Boot applications using separate schemas or databases. Explore Hibernate support, tenant resolvers, and design considerations for scalable SaaS systems.
---
**Multi-tenancy** is a key architectural pattern in Software as a Service (SaaS) platforms. It allows a single application instance to serve multiple customers (tenants) while logically isolating their data.

In this post, we’ll explore how to implement **multi-tenancy in Spring Boot** using **Hibernate**, **Spring Data JPA**, and **TenantContext resolution** strategies. We'll cover schema-based and database-based multi-tenancy, tenant identification, and best practices for secure and scalable design.

---

#### What is Multi-Tenancy?

Multi-tenancy enables a single application to serve multiple clients with data isolation. There are three main strategies:

1. **Shared Database, Shared Schema** (Row-level isolation)
2. **Shared Database, Separate Schemas** (Schema-per-tenant)
3. **Separate Database per Tenant** (Full isolation)

Each has trade-offs in terms of complexity, security, and performance.

---

#### Choosing a Multi-Tenant Strategy

| Strategy | Isolation | Complexity | Use Case |
|----------|-----------|------------|----------|
| Shared schema | Low | Low | Small-scale apps |
| Schema per tenant | Medium | Medium | Moderate isolation |
| Database per tenant | High | High | Enterprise-level SaaS |

For this guide, we'll focus on **schema-per-tenant** and **database-per-tenant** approaches using Hibernate.

---

#### Setting Up Dependencies

Add Hibernate and Spring Data JPA to your `pom.xml`:

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

#### Schema-Based Multi-Tenancy Configuration

Use Hibernate's multi-tenant support to switch schemas dynamically.

Enable `MultiTenancyStrategy.SCHEMA`:

```java
@Configuration
public class HibernateConfig {

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(
        DataSource dataSource,
        MultiTenantConnectionProvider connectionProvider,
        CurrentTenantIdentifierResolver tenantIdentifierResolver) {

        Map<String, Object> properties = new HashMap<>();
        properties.put(Environment.MULTI_TENANT, MultiTenancyStrategy.SCHEMA);
        properties.put(Environment.MULTI_TENANT_CONNECTION_PROVIDER, connectionProvider);
        properties.put(Environment.MULTI_TENANT_IDENTIFIER_RESOLVER, tenantIdentifierResolver);

        return new LocalContainerEntityManagerFactoryBeanBuilder()
            .dataSource(dataSource)
            .packages("com.example.model")
            .persistenceUnit("tenantPU")
            .properties(properties)
            .build();
    }
}
```

---

#### Tenant Identifier Resolver

Create a context holder and resolver to identify the current tenant.

**TenantContext.java**

```java
public class TenantContext {
private static final ThreadLocal<String> CURRENT_TENANT = new ThreadLocal<>();

    public static void setTenantId(String tenantId) {
        CURRENT_TENANT.set(tenantId);
    }

    public static String getTenantId() {
        return CURRENT_TENANT.get();
    }

    public static void clear() {
        CURRENT_TENANT.remove();
    }
}
```

**TenantIdentifierResolver.java**

```java
@Component
public class TenantIdentifierResolver implements CurrentTenantIdentifierResolver {
@Override
public String resolveCurrentTenantIdentifier() {
return Optional.ofNullable(TenantContext.getTenantId()).orElse("default");
}

    @Override
    public boolean validateExistingCurrentSessions() {
        return true;
    }
}
```

---

#### MultiTenantConnectionProvider Implementation

```java
@Component
public class SchemaMultiTenantConnectionProvider implements MultiTenantConnectionProvider {

    @Autowired
    private DataSource dataSource;

    @Override
    public Connection getConnection(String tenantIdentifier) throws SQLException {
        Connection connection = dataSource.getConnection();
        connection.setSchema(tenantIdentifier);
        return connection;
    }

    @Override
    public Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }

    // Other overridden methods...
}
```

This allows Hibernate to switch schemas per request.

---

#### Intercepting and Setting Tenant Context

Use a filter or interceptor to set the tenant based on the request:

```java
@Component
public class TenantFilter extends OncePerRequestFilter {

    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
        throws ServletException, IOException {

        String tenantId = request.getHeader("X-Tenant-ID");
        if (tenantId != null) {
            TenantContext.setTenantId(tenantId);
        }

        try {
            filterChain.doFilter(request, response);
        } finally {
            TenantContext.clear();
        }
    }
}
```

This ensures tenant resolution is request-scoped.

---

#### Database-Per-Tenant Strategy (Optional)

For full isolation, each tenant can have a separate database. You'll need a dynamic `DataSourceRouter`.

```java
public class MultiTenantDataSourceRouter extends AbstractRoutingDataSource {
@Override
protected Object determineCurrentLookupKey() {
return TenantContext.getTenantId();
}
}
```

Preload or lazily initialize `DataSource` instances per tenant and map them accordingly.

---

#### Best Practices

- Use **Flyway** or **Liquibase** to manage tenant schema evolution
- Cache tenant-specific metadata (e.g., DB credentials or schema names)
- Validate `tenantId` before using it to prevent injection attacks
- Encrypt and isolate tenant data where possible
- Monitor tenant-specific metrics for resource usage and scaling

---

#### Conclusion

Implementing **multi-tenancy in Spring Boot** with Spring Data and Hibernate gives you a scalable foundation for SaaS development. Whether you’re using schema-based or database-per-tenant isolation, Spring’s extensible architecture lets you adapt to your organization’s requirements.

With a clear strategy for tenant resolution, data isolation, and security, you can confidently build apps that serve multiple clients without compromising on performance or maintainability.
