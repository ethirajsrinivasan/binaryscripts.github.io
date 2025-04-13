---
layout: post
title: Securing Spring Boot Applications with JWT and OAuth2
subtitle: Implement robust authentication and authorization in Spring Boot using JWT and OAuth2
categories: SpringBoot
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: [Spring Boot, JWT, OAuth2, Security, Authentication, Authorization, Java]
excerpt: Learn how to secure Spring Boot applications using JWT and OAuth2. This guide covers token-based authentication, access control, role-based authorization, and integration with third-party identity providers.
---
Security is a critical concern in any modern web application. With the rise of microservices and distributed architectures, token-based security models like **OAuth2** and **JWT (JSON Web Tokens)** have become the standard for **stateless authentication and authorization**.

This guide walks through securing Spring Boot applications using **OAuth2 protocols** and **JWT tokens**. Whether you're building APIs or full-stack apps, this setup offers flexible and scalable identity and access management.

---

#### Why Use JWT and OAuth2?

- **OAuth2** provides a standardized framework for token-based authentication and authorization.
- **JWT** is a compact, self-contained token format used to pass user information securely.

Benefits include:
- Stateless session management (no server-side session)
- Easy integration with third-party identity providers (Google, Okta, Auth0)
- Role and scope-based access control
- Efficient token validation with no DB calls

---

#### Add Dependencies

Include the following in your `pom.xml` for Spring Security and OAuth2 support:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

Optional for JWT customization:

```xml
<dependency>
<groupId>org.springframework.security</groupId>
<artifactId>spring-security-oauth2-jose</artifactId>
</dependency>
```

---

#### Configuring JWT-Based Authentication

Enable JWT-based authentication in `application.yml`:

```yml
spring:
security:
oauth2:
resourceserver:
jwt:
issuer-uri: https://auth.myapp.com/realms/myrealm
```

Spring Boot will automatically validate JWT tokens using the **issuer URI’s** discovery endpoint (typically a Keycloak, Auth0, or Okta server).

---

#### Define Security Configuration

Create a `SecurityConfig` class to enforce security rules:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt);
        return http.build();
    }
}
```

Use **role-based** or **scope-based** access control with Spring Expression Language (SpEL).

---

#### Accessing User Details from JWT

You can extract claims from the token in your controller:

```java
@GetMapping("/profile")
public Map<String, Object> getUserProfile(@AuthenticationPrincipal Jwt jwt) {
return Map.of(
"username", jwt.getClaim("preferred_username"),
"email", jwt.getClaim("email")
);
}
```

This is especially useful for personalizing user-facing services or logging audit trails.

---

#### OAuth2 Login (Client Side)

If you want to support **OAuth2 login** (e.g., via Google or GitHub), configure `application.yml`:

```yml
spring:
security:
oauth2:
client:
registration:
google:
client-id: YOUR_CLIENT_ID
client-secret: YOUR_SECRET
scope: openid, profile, email
```

Enable login in your security config:

```java
http
.authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
.oauth2Login();
```

This enables automatic redirection to the provider’s login page and token retrieval.

---

#### Token Introspection (Optional)

If using opaque tokens instead of JWT, enable introspection:

```yml
spring:
security:
oauth2:
resourceserver:
opaque-token:
introspection-uri: https://auth.myapp.com/oauth2/introspect
client-id: my-client-id
client-secret: my-client-secret
```

This performs a live check against the authorization server.

---

#### Protecting Endpoints with Scopes

Use method-level security annotations:

```java
@PreAuthorize("hasAuthority('SCOPE_read:users')")
@GetMapping("/users")
public List<UserDTO> getUsers() {
return userService.getAll();
}
```

Add `@EnableMethodSecurity` to your configuration:

```java
@Configuration
@EnableMethodSecurity
public class MethodSecurityConfig {}
```

---

#### JWT Customization and Claims Mapping

Use a custom `JwtAuthenticationConverter` to map roles or permissions:

```java
public class CustomJwtAuthenticationConverter implements Converter<Jwt, AbstractAuthenticationToken> {
@Override
public AbstractAuthenticationToken convert(Jwt jwt) {
Collection<GrantedAuthority> authorities = extractAuthorities(jwt);
return new JwtAuthenticationToken(jwt, authorities);
}
}
```

Register it in your security config if you need fine-grained control.

---

#### Best Practices

- Use HTTPS everywhere to protect tokens in transit
- Keep JWTs short-lived (e.g., 15 minutes)
- Use refresh tokens to renew access securely
- Validate audience and issuer fields on tokens
- Rotate signing keys regularly

---

#### Conclusion

Securing your Spring Boot applications with **JWT and OAuth2** provides a scalable, modern, and stateless approach to authentication and authorization. Whether you're integrating with external providers or managing your own identity system, Spring Boot’s support for these standards makes it straightforward to implement.

Mastering token-based security helps you build applications that are both **secure** and **future-ready**.
