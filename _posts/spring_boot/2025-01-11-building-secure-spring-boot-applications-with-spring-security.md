---
layout: post
title: Building Secure Spring Boot Applications with Spring Security
subtitle: Secure your Spring Boot apps using authentication, authorization, and modern Spring Security practices
categories: SpringBoot
tags: [Spring Boot, Java, Spring Security, JWT, OAuth2, Authentication, Authorization]
excerpt: Learn how to secure your Spring Boot applications using Spring Security. Explore configurations for authentication, role-based access, JWT integration, CSRF protection, and best practices for modern web security.
---
Security is one of the most critical aspects of web application development. Without proper safeguards, your app is vulnerable to unauthorized access, data leaks, and injection attacks. Fortunately, **Spring Security** offers a robust and customizable security framework that integrates seamlessly with **Spring Boot**.

In this post, we’ll explore how to build **secure Spring Boot applications** using Spring Security. We’ll cover authentication, authorization, JWT integration, CSRF protection, custom filters, and essential best practices for modern application security.

---

#### Getting Started with Spring Security

Add the dependency in your `pom.xml`:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

By default, Spring Security:
- Secures all endpoints
- Requires basic authentication
- Auto-generates a default password at startup

---

#### Configuring Basic Authentication

You can define in-memory users for quick testing:

```yml
spring:
security:
user:
name: admin
password: secret
roles: ADMIN
```

Or configure it programmatically:

```java
@Bean
public UserDetailsService userDetailsService() {
UserDetails user = User.withUsername("admin")
.password(passwordEncoder().encode("secret"))
.roles("ADMIN")
.build();
return new InMemoryUserDetailsManager(user);
}
```

Always encode passwords using a `PasswordEncoder`:

```java
@Bean
public PasswordEncoder passwordEncoder() {
return new BCryptPasswordEncoder();
}
```

---

#### Defining Authorization Rules

You can secure endpoints using HTTP method and path patterns:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
http.csrf().disable()
.authorizeHttpRequests(auth -> auth
.requestMatchers("/admin/**").hasRole("ADMIN")
.requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
.anyRequest().authenticated()
)
.httpBasic();
return http.build();
}
```

Use `@PreAuthorize` or `@Secured` annotations for method-level access control:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { ... }
```

---

#### JWT-Based Authentication

For stateless APIs, implement **JWT (JSON Web Token)** authentication.

1. Client logs in and receives a signed JWT
2. All subsequent requests include the token in headers
3. Server validates and extracts user information

JWT login controller:

```java
@PostMapping("/auth/login")
public ResponseEntity<?> authenticate(@RequestBody LoginRequest loginRequest) {
Authentication auth = authManager.authenticate(
new UsernamePasswordAuthenticationToken(loginRequest.getUsername(), loginRequest.getPassword())
);
String jwt = jwtService.generateToken(auth);
return ResponseEntity.ok(new JwtResponse(jwt));
}
```

JWT filter for request validation:

```java
public class JwtAuthFilter extends OncePerRequestFilter {
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
throws ServletException, IOException {
String token = extractToken(request);
if (token != null && jwtService.validateToken(token)) {
UsernamePasswordAuthenticationToken auth = jwtService.getAuthentication(token);
SecurityContextHolder.getContext().setAuthentication(auth);
}
chain.doFilter(request, response);
}
}
```

Add the filter to your `SecurityFilterChain`.

---

#### Enabling CSRF Protection

For stateful web applications, **Cross-Site Request Forgery (CSRF)** protection is enabled by default.

If you’re building APIs or using JWT, you can disable it:

```java
http.csrf().disable()
```

For web forms, Spring Security automatically injects a CSRF token:

```html
<input type="hidden" name="_csrf" value="${_csrf.token}"/>
```

---

#### Configuring CORS

If your frontend runs on a different origin (e.g., React or Angular), enable **CORS (Cross-Origin Resource Sharing)**:

```java
@Bean
public WebMvcConfigurer corsConfigurer() {
return new WebMvcConfigurer() {
public void addCorsMappings(CorsRegistry registry) {
registry.addMapping("/**")
.allowedOrigins("http://localhost:3000")
.allowedMethods("GET", "POST", "PUT", "DELETE");
}
};
}
```

---

#### Customizing AuthenticationEntryPoint

Customize how Spring handles unauthorized access:

```java
@Component
public class CustomAuthEntryPoint implements AuthenticationEntryPoint {
public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException)
throws IOException {
response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
response.getWriter().write("Unauthorized access");
}
}
```

Then wire it into your security config:

```java
http.exceptionHandling().authenticationEntryPoint(customAuthEntryPoint);
```

---

#### Security Best Practices

- **Encrypt passwords** using BCrypt or Argon2
- Use **HTTPS** for all communication
- Apply **rate limiting** and brute-force protection
- Avoid exposing stack traces and internal messages in error responses
- Store JWT secrets securely (e.g., in Vault or env variables)
- Sanitize input to avoid injection attacks
- Keep dependencies up to date

---

#### Conclusion

Securing Spring Boot applications requires a holistic approach — from endpoint protection to token validation and CORS configuration. With **Spring Security**, developers have a powerful and flexible toolkit to enforce both authentication and fine-grained authorization.

By combining best practices with modern standards like **JWT**, you can confidently build secure, production-ready applications.
