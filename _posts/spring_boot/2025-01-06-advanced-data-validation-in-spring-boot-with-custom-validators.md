---
layout: post
title: Advanced Data Validation in Spring Boot with Custom Validators
subtitle: Implement robust and reusable validation logic in Spring Boot using annotations and custom validators
categories: Spring Boot
tags: [Java, Spring Boot, Validation, Custom Validator, Hibernate Validator, Bean Validation]
excerpt: Master advanced data validation in Spring Boot by creating custom validators, combining annotations, and improving data integrity across layers with reusable validation logic.
---



Data validation is a critical part of building secure, reliable, and user-friendly applications. Spring Boot provides robust support for **bean validation** using **Hibernate Validator**, but complex business logic often requires more than built-in constraints like `@NotNull` or `@Email`.

In this post, we’ll explore **advanced data validation techniques** in Spring Boot, including:
- Creating custom validation annotations
- Reusing constraint logic across DTOs and entities
- Composing constraints
- Handling validation groups and dynamic scenarios

---

#### Enabling Validation in Spring Boot

Spring Boot integrates with **Hibernate Validator** (the reference implementation of Bean Validation 2.0). To get started, include the dependency (already bundled with Spring Boot Starter Web):

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Activate validation by annotating controller methods with `@Valid` or `@Validated`:

```java
@PostMapping("/register")
public ResponseEntity<?> register(@Valid @RequestBody UserDTO dto) {
// registration logic
}
```

---

#### Creating a Custom Validator Annotation

Built-in constraints are limited for domain-specific rules. Let’s say we want to validate that a username is unique in the database.

**Step 1: Create the Annotation**

```java
@Target({ ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueUsernameValidator.class)
public @interface UniqueUsername {
String message() default "Username is already taken";
Class<?>[] groups() default {};
Class<? extends Payload>[] payload() default {};
}
```

**Step 2: Implement the Validator Logic**

```java
public class UniqueUsernameValidator implements ConstraintValidator<UniqueUsername, String> {

    @Autowired
    private UserRepository userRepository;

    @Override
    public boolean isValid(String username, ConstraintValidatorContext context) {
        return !userRepository.existsByUsername(username);
    }
}
```

Spring will now inject your repository and validate this constraint during form submission.

---

#### Composing Multiple Constraints

You can combine standard and custom annotations for more expressive validation:

```java
public class UserDTO {

    @NotBlank
    @Size(min = 4, max = 20)
    @UniqueUsername
    private String username;

    @Email
    @NotBlank
    private String email;
}
```

Validation will cascade and fail-fast depending on your configuration.

---

#### Field-Level vs Class-Level Validation

Use **class-level validation** when validation logic depends on multiple fields — for example, matching passwords.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PasswordMatchesValidator.class)
public @interface PasswordMatches {
String message() default "Passwords do not match";
Class<?>[] groups() default {};
Class<? extends Payload>[] payload() default {};
}
```

```java
public class PasswordMatchesValidator implements ConstraintValidator<PasswordMatches, RegisterDTO> {
public boolean isValid(RegisterDTO dto, ConstraintValidatorContext context) {
return dto.getPassword().equals(dto.getConfirmPassword());
}
}
```

Apply it like this:

```java
@PasswordMatches
public class RegisterDTO {
private String password;
private String confirmPassword;
}
```

---

#### Using Validation Groups

Validation groups allow you to run different validations depending on the context (e.g., create vs update).

```java
public interface OnCreate {}
public interface OnUpdate {}

@NotBlank(groups = OnCreate.class)
private String name;
```

Then specify the group in your controller:

```java
@Validated(OnCreate.class)
@PostMapping("/create")
public void create(@RequestBody UserDTO dto) { ... }
```

This pattern keeps your DTOs flexible without duplicating code.

---

#### Customizing Validation Error Responses

Handle validation errors globally using `@RestControllerAdvice`:

```java
@RestControllerAdvice
public class ValidationExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }
}
```

This provides structured error messages to clients for better UX.

---

#### Advanced Tips

- Use `ConstraintValidatorContext` to customize messages dynamically
- Add dynamic properties via `@Constraint(validatedBy =...)` to reuse logic
- Use `@Validated` on service layers for method-level validation
- Combine annotations with `@Target({ElementType.FIELD, ElementType.PARAMETER})` for flexibility
- Avoid calling services in validators if not necessary — cache if possible

---

#### Conclusion

Data validation is more than just checking for nulls. With **custom annotations**, **class-level constraints**, and **validation groups**, Spring Boot gives you powerful tools to ensure your data integrity and business rules are enforced cleanly.

By mastering advanced validation techniques, you can simplify your controllers, reduce bugs, and enforce contracts effectively across layers of your application.
