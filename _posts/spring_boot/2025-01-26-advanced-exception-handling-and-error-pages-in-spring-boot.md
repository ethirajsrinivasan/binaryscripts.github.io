---
layout: post
title: Advanced Exception Handling and Error Pages in Spring Boot
subtitle: Master structured error handling and user-friendly error pages in Spring Boot applications
categories: SpringBoot
tags: [Java, Spring Boot, Exception Handling, Error Pages, REST, ControllerAdvice, Global Error Handling]
excerpt: Learn how to build robust Spring Boot applications with advanced exception handling and custom error pages. Implement global handlers, structured JSON responses, and user-friendly error views.
---
Handling exceptions effectively is critical to building **robust and user-friendly** Spring Boot applications. Whether you're building REST APIs or web applications, users and clients need consistent, informative, and secure error responses.

This post dives into **advanced exception handling** in Spring Boot, covering:
- Global error handling with `@ControllerAdvice`
- Structured error responses for APIs
- Custom error views for web apps
- HTTP status mapping
- Logging and diagnostics

---

#### Default Error Handling in Spring Boot

Out of the box, Spring Boot provides a global error controller (`BasicErrorController`) that returns:
- JSON responses for REST clients
- Whitelabel error pages for browser requests

Example default response:

```json
{
"timestamp": "2024-11-16T09:23:18.326+00:00",
"status": 404,
"error": "Not Found",
"path": "/api/data"
}
```

While useful, it lacks structure, control, and branding. Let’s customize it for better UX and observability.

---

#### Creating a Global Exception Handler

Use `@RestControllerAdvice` to intercept exceptions thrown across your app:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex, HttpServletRequest request) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            request.getRequestURI()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex, HttpServletRequest request) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            request.getRequestURI()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

Create a standard error response model:

```java
public class ErrorResponse {
private int status;
private String message;
private String path;

    // constructor, getters, setters
}
```

This ensures a consistent structure across all API errors.

---

#### Handling Validation Errors

Spring automatically throws `MethodArgumentNotValidException` when validation fails. Handle it like this:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<?> handleValidationErrors(MethodArgumentNotValidException ex) {
Map<String, String> errors = new HashMap<>();
ex.getBindingResult().getFieldErrors().forEach(error ->
errors.put(error.getField(), error.getDefaultMessage()));
return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
}
```

This sends field-level error messages that frontend clients can use directly.

---

#### Custom Error Pages for Web Applications

For MVC or Thymeleaf-based apps, you can define custom HTML error pages:

1. Place your templates in `/src/main/resources/templates/error/`

2. Use the name pattern: `error/404.html`, `error/500.html`

Spring will resolve them based on the HTTP status code:

```html
<!-- templates/error/404.html -->
<!DOCTYPE html>
<html>
<head><title>Page Not Found</title></head>
<body>
<h1>Oops! 404</h1>
<p>The page you are looking for does not exist.</p>
</body>
</html>
```

This allows you to match your site’s branding and improve the user experience.

---

#### Overriding Default
