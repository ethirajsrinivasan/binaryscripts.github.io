---
layout: post
title: Securing Java Applications with Keycloak
subtitle: Integrate Keycloak with Java apps for robust authentication and authorization
categories: Java
tags: [Java, Keycloak, OAuth2, Authentication, Security, Spring Security, OpenID]
excerpt: Learn how to secure Java applications using Keycloak. Set up SSO, OAuth2 flows, role-based access control, and JWT validation with Spring Boot and Jakarta EE integration.
---



Security is a critical part of any enterprise application — but implementing authentication and authorization from scratch can be time-consuming and error-prone. **Keycloak** is an open-source identity and access management tool that simplifies user login, role-based access, and integration with OAuth2 and OpenID Connect.

In this post, you’ll learn how to integrate **Keycloak with Java applications** using **Spring Boot**, configure login flows, secure endpoints, and enforce role-based access with minimal effort.

---

#### What is Keycloak?

**Keycloak** is a powerful identity provider (IdP) with support for:
- Single Sign-On (SSO)
- OAuth2 and OpenID Connect
- Social login (Google, Facebook, GitHub)
- Role-based access control (RBAC)
- LDAP and Active Directory integration

It provides a web UI for managing users, roles, and clients, and can be deployed as a standalone server or embedded service.

---

#### Keycloak Architecture Overview

- **Realm**: A namespace for users, clients, roles
- **Client**: An application that connects to Keycloak for auth
- **User**: Authenticated entity with credentials
- **Role**: Permissions assigned to users or clients
- **Token**: Access tokens issued upon successful login

When a Java app integrates with Keycloak:
1. The app redirects users to Keycloak login
2. Keycloak authenticates and returns a JWT
3. The app verifies the token and grants access

---

#### Step 1: Start a Local Keycloak Server

Use the Keycloak Docker image:

```bash
docker run -p 8080:8080 \
-e KEYCLOAK_ADMIN=admin \
-e KEYCLOAK_ADMIN_PASSWORD=admin \
quay.io/keycloak/keycloak:22.0.3 \
start-dev
```

Open the admin console at `http://localhost:8080/admin`.

---

#### Step 2: Create a Realm and Client

1. Create a new realm called `demo`.
2. Add a client (e.g., `java-app`) with `confidential` access type.
3. Set valid redirect URIs, e.g., `http://localhost:8081/*`.
4. Save the client credentials.

---

#### Step 3: Integrate with a Spring Boot App

Add the Keycloak and Spring Boot dependencies:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

---

#### Step 4: Configure application.yml

```yml
spring:
security:
oauth2:
resourceserver:
jwt:
issuer-uri: http://localhost:8080/realms/demo
```

This will automatically fetch the JWKS endpoint for token validation.

---

#### Step 5: Secure Endpoints

You can now protect endpoints using role-based access:

```java
@RestController
public class HelloController {

    @GetMapping("/public")
    public String publicEndpoint() {
        return "This is a public endpoint";
    }

    @GetMapping("/secure")
    @PreAuthorize("hasAuthority('ROLE_user')")
    public String secureEndpoint() {
        return "This is a secure endpoint";
    }
}
```

Enable method-level security:

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig {
// ...
}
```

---

#### Step 6: Testing the Integration

1. Get a token using Keycloak’s token endpoint:

```bash
curl --location --request POST 'http://localhost:8080/realms/demo/protocol/openid-connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'client_id=java-app' \
--data-urlencode 'client_secret=YOUR_SECRET' \
--data-urlencode 'username=testuser' \
--data-urlencode 'password=password'
```

2. Use the access token to call the secure endpoint:

```bash
curl -H "Authorization: Bearer $TOKEN" http://localhost:8081/secure
```

---

#### Managing Roles in Keycloak

Assign roles directly to users or client scopes. Keycloak includes these roles in the access token’s `realm_access` claim.

Spring automatically maps them to `ROLE_*` authorities.

---

#### Best Practices

- Use HTTPS in production to protect tokens
- Rotate client secrets regularly
- Leverage refresh tokens for long sessions
- Use mTLS for backend-to-backend communication
- Enable session expiration and audit logging in Keycloak

---

#### Conclusion

Keycloak makes it easy to secure Java applications with **minimal code changes** and **powerful features** like SSO, role-based access control, and federated identity.

By offloading authentication and token management to Keycloak, you can focus on writing business logic while maintaining enterprise-grade security and flexibility.
