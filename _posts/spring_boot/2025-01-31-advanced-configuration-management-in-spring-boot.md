---
layout: post
title: Advanced Configuration Management in Spring Boot
subtitle: Manage complex application properties with profiles, external configs, encryption, and centralized config servers
categories: Spring Boot
tags: [Spring Boot, Configuration, Profiles, Spring Cloud Config, YAML, Encryption]
excerpt: Learn how to manage complex configuration in Spring Boot using profiles, property injection, YAML, environment variables, and Spring Cloud Config for centralized management.
---



Managing configuration is critical in modern applications. As environments grow more complex — from local development to cloud deployments — applications need flexible, secure, and scalable ways to manage configuration.

Spring Boot excels at **configuration management**, offering support for property files, YAML, profiles, environment variables, command-line arguments, and even externalized configuration services like **Spring Cloud Config**.

In this post, we’ll dive into **advanced configuration techniques in Spring Boot**, including profile-based configs, property injection, hierarchical overrides, encrypted values, and centralized configuration servers.

---

#### Configuration Sources in Spring Boot

Spring Boot automatically loads configuration from multiple sources in a well-defined order:

1. `application.properties` / `application.yml`
2. Profile-specific files: `application-dev.yml`
3. Environment variables
4. Command-line arguments
5. Config data from Spring Cloud Config
6. System properties and OS-level environment variables

This layered approach provides **flexibility** and **overriding capabilities** depending on deployment environments.

---

#### Using YAML for Structured Configuration

Instead of flat `.properties` files, Spring Boot supports structured YAML files.

```yml
server:
port: 8081

spring:
datasource:
url: jdbc:mysql://localhost:3306/app_db
username: root
password: secret
```

Use dot notation in `@Value` to access nested properties:

```java
@Value("${spring.datasource.url}")
private String datasourceUrl;
```

---

#### Type-Safe Configuration with @ConfigurationProperties

Avoid scattered `@Value` annotations by binding configuration to a POJO:

```java
@Configuration
@ConfigurationProperties(prefix = "app")
public class AppProperties {
private String name;
private int maxConnections;
}
```

And in `application.yml`:

```yml
app:
name: MySpringApp
maxConnections: 50
```

This approach provides better IDE support, type safety, and testability.

---

#### Profile-Specific Configuration

Spring Boot supports environment-specific configuration via **profiles**.

Create separate files like:

- `application-dev.yml`
- `application-prod.yml`

Activate a profile:

```yml
spring:
profiles:
active: dev
```

Or via command line:

```bash
java -jar app.jar --spring.profiles.active=prod
```

You can even conditionally load beans:

```java
@Profile("dev")
@Bean
public DataSource devDataSource() { ... }
```

---

#### Environment Variable and Command-Line Overrides

Spring Boot automatically maps environment variables using relaxed binding. For example:

```bash
export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost/prod_db
```

Or use a launch argument:

```bash
java -jar app.jar --server.port=9090
```

This is perfect for containerized environments and CI/CD pipelines.

---

#### Secure Configuration with Encrypted Properties

Use tools like **Jasypt** to encrypt sensitive values like passwords and API keys.

Add the dependency:

```xml
<dependency>
<groupId>com.github.ulisesbocchio</groupId>
<artifactId>jasypt-spring-boot-starter</artifactId>
<version>3.0.5</version>
</dependency>
```

Encrypted property:

```yml
spring:
datasource:
password: ENC(kjdsf239u3klsdf==)
```

Then configure the encryption key via env variable:

```bash
export JASYPT_ENCRYPTOR_PASSWORD=mysecretkey
```

---

#### Externalizing Configuration with Spring Cloud Config

For centralized config management across services, use **Spring Cloud Config**.

It supports:
- Git-backed configuration repositories
- Dynamic refresh of properties (`@RefreshScope`)
- Multi-environment management

**Client setup:**

```yml
spring:
config:
import: optional:configserver:http://localhost:8888
```

**Server setup:**

```yml
spring:
cloud:
config:
server:
git:
uri: https://github.com/my-org/config-repo
```

Clients can now load `application-dev.yml` or `service-name-prod.yml` from Git!

---

#### Refreshing Configuration at Runtime

Use `@RefreshScope` to reload beans dynamically without restarting the app:

```java
@RefreshScope
@Component
public class ConfigurableService {
@Value("${app.feature.enabled}")
private boolean featureToggle;
}
```

Trigger a refresh using actuator:

```bash
curl -X POST http://localhost:8080/actuator/refresh
```

Useful for toggling features or updating limits on the fly.

---

#### Best Practices for Configuration Management

- Use profiles for clean environment separation
- Avoid hardcoding secrets — inject via environment variables or Vault
- Use `@ConfigurationProperties` for maintainability
- Use `.yml` over `.properties` for nested configs
- Centralize configuration using Spring Cloud Config for microservices
- Encrypt sensitive values at rest and in transit

---

#### Conclusion

Advanced configuration management is essential for building resilient, environment-aware Spring Boot applications. By leveraging Spring Boot’s robust support for **profiles**, **external properties**, **encryption**, and **config servers**, you can keep your applications flexible, secure, and production-ready.

Mastering these configuration techniques allows your team to deploy, scale, and manage Java applications with confidence — across development, staging, and production environments.
