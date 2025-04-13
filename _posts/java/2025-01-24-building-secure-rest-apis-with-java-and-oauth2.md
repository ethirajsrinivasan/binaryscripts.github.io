---
layout: post
title: Building Secure REST APIs with Java and OAuth2
subtitle: Secure your RESTful Java APIs using OAuth2 authentication and authorization best practices
categories: Java
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: [Java, REST, OAuth2, Security, Spring Boot, API Security]
excerpt: Learn how to secure REST APIs in Java using OAuth2 protocols. Understand token-based authentication, access scopes, and integration with Spring Security for enterprise-grade security.
---
APIs power the modern web — from microservices to mobile backends. But with growing threats and compliance requirements, **security is no longer optional**. OAuth2 has become the de-facto standard for secure API authentication and authorization.

In this guide, we’ll explore how to build and secure REST APIs in Java using **OAuth2**. We’ll cover concepts like access tokens, scopes, client credentials, and how to implement OAuth2 flows using **Spring Boot and Spring Security**.

---

#### What is OAuth2?

**OAuth2** is an open standard for token-based authentication. It allows third-party applications to access resources on behalf of a user without sharing credentials.

Key components:
- **Resource Owner:** The user
- **Client:** The app requesting access
- **Authorization Server:** Issues tokens
- **Resource Server:** Validates and serves data

Common OAuth2 flows:
- Authorization Code (used in browser-based apps)
- Client Credentials (used in machine-to-machine communication)
- Password Grant (legacy, not recommended)

---

#### Add OAuth2 Dependencies

Using Spring Boot? Add these to your `pom.xml`:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
<groupId>org.springframework.security</groupId>
<artifactId>spring-security-oauth2-jose</artifactId>
</dependency>
```

These enable JWT decoding, token validation, and role-based access.

---

#### Secure REST APIs with Spring Security

Here’s how to enable your REST endpoints to require a **JWT token**:

```java
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt);
        return http.build();
    }
}
```

This config:
- Allows public access to `/public/**`
- Secures all other endpoints
- Validates JWT tokens

---

#### Configuring the Authorization Server

You’ll need a provider like:
- **Keycloak**
- **Auth0**
- **Okta**
- **AWS Cognito**

They issue tokens after successful authentication. Spring automatically validates incoming tokens based on the issuer and public keys:

```yml
spring:
security:
oauth2:
resourceserver:
jwt:
issuer-uri: https://auth.myapp.com/realms/app
```

---

#### Role-Based Access Control with Scopes

Secure endpoints based on user roles or scopes:

```java
@GetMapping("/admin")
@PreAuthorize("hasAuthority('SCOPE_admin')")
public String adminEndpoint() {
return "Restricted admin data";
}
```

To use this, your tokens must contain a `scope` claim.

---

#### Testing OAuth2 Secured Endpoints

Use Postman or `curl` to simulate real-world calls:

```bash
curl -H "Authorization: Bearer <access_token>" http://localhost:8080/api/private
```

In tests, mock the JWT token:

```java
@Test
@WithMockJwtAuth(authorities = {"SCOPE_user"})
void shouldAccessPrivateEndpoint() {
// perform GET and assert
}
```

Or use Spring Security's `@WithMockUser` for simple mocking.

---

#### Refresh Tokens and Expiry

Tokens typically expire after 15–60 minutes. Use refresh tokens to get a new access token without re-authenticating.

Make sure refresh tokens are:
- Stored securely
- Invalidated on logout
- Issued only to trusted clients

---

#### Best Practices

- Always use **HTTPS** — never send tokens over plain HTTP
- Validate JWT signature and expiration
- Limit token lifetime
- Use strong client secrets
- Log suspicious access patterns

---

#### Conclusion

Building secure REST APIs with Java is straightforward when using **OAuth2** and **Spring Security**. Whether you’re protecting internal microservices or public APIs, OAuth2 provides a robust, flexible, and widely adopted standard.

By implementing access tokens, scopes, and role-based security, you can build systems that are both powerful and protected.

