---
layout: post
title: Spring Boot Reactive Testing with StepVerifier
subtitle: Test reactive streams effectively in Spring Boot using StepVerifier and Project Reactor
categories: Spring Boot
tags: [Spring Boot, Reactive, WebFlux, Testing, StepVerifier, Project Reactor]
excerpt: Learn how to test Spring Boot reactive applications with StepVerifier. Validate Mono and Flux streams using assertions, time manipulation, and best practices for non-blocking tests.
---



Reactive programming is revolutionizing how Java applications are built. Spring Boot, with **WebFlux** and **Project Reactor**, allows developers to create fully non-blocking, reactive applications that scale with fewer resources.

But testing reactive code introduces unique challenges. Traditional testing techniques often fail to validate asynchronous, time-sensitive behavior properly. This is where **StepVerifier**, part of the Reactor Test module, becomes essential.

In this blog, you’ll learn how to test reactive streams in **Spring Boot** using **StepVerifier**, ensuring your `Mono` and `Flux` pipelines behave as expected — even under delay, error, or infinite stream conditions.

---

#### Setup: Include Dependencies

Make sure you’ve added the right dependencies in your `pom.xml` or `build.gradle`.

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
<groupId>io.projectreactor</groupId>
<artifactId>reactor-test</artifactId>
<scope>test</scope>
</dependency>
```

This provides access to **Reactor Core** and **StepVerifier**, the main tool for reactive assertions.

---

#### Basic Mono Testing with StepVerifier

Let’s test a method that returns a `Mono<String>`:

```java
public Mono<String> greet(String name) {
return Mono.just("Hello, " + name);
}
```

The test case:

```java
@Test
void testMono() {
Mono<String> result = service.greet("Alice");

    StepVerifier.create(result)
        .expectNext("Hello, Alice")
        .expectComplete()
        .verify();
}
```

You can also set timeouts:

```java
.verify(Duration.ofMillis(500));
```

---

#### Testing Flux with Multiple Values

For streams emitting multiple elements:

```java
public Flux<Integer> numbers() {
return Flux.range(1, 3);
}
```

Test it with:

```java
@Test
void testFlux() {
Flux<Integer> numbers = service.numbers();

    StepVerifier.create(numbers)
        .expectNext(1)
        .expectNext(2)
        .expectNext(3)
        .expectComplete()
        .verify();
}
```

Or in a shorter way:

```java
.expectNextSequence(List.of(1, 2, 3))
```

---

#### Testing Errors in Reactive Streams

Let’s test a `Mono` that emits an error:

```java
public Mono<String> findById(String id) {
return Mono.error(new RuntimeException("Not found"));
}
```

Verify it as:

```java
@Test
void testMonoError() {
Mono<String> mono = service.findById("123");

    StepVerifier.create(mono)
        .expectErrorMatches(throwable -> 
            throwable instanceof RuntimeException &&
            throwable.getMessage().equals("Not found"))
        .verify();
}
```

---

#### Testing Delays and Virtual Time

For time-based streams, use **virtual time** to simulate delays:

```java
public Flux<Long> delayedStream() {
return Flux.interval(Duration.ofSeconds(1)).take(3);
}
```

Test without waiting in real time:

```java
@Test
void testWithVirtualTime() {
StepVerifier.withVirtualTime(() -> service.delayedStream())
.thenAwait(Duration.ofSeconds(3))
.expectNext(0L, 1L, 2L)
.expectComplete()
.verify();
}
```

Virtual time is ideal for long-running or scheduled streams.

---

#### Combining with Mockito or WebTestClient

You can mock services returning `Mono`/`Flux` and verify them:

```java
when(repo.findByUsername("alice")).thenReturn(Mono.just(user));
```

And test reactive controllers with `WebTestClient`:

```java
@Test
void testReactiveEndpoint() {
webTestClient.get().uri("/api/users/alice")
.exchange()
.expectStatus().isOk()
.expectBody()
.jsonPath("$.username").isEqualTo("alice");
}
```

`WebTestClient` is the reactive equivalent of `MockMvc`.

---

#### Common Gotchas

- **Don’t block!** Never use `.block()` or `.subscribe()` in tests unless you’re explicitly testing a blocking call.
- **Always assert terminal signals** like `expectComplete()` or `expectError()`.
- For hot publishers, consider using `ConnectableFlux` and `.autoConnect()` to simulate subscriptions.

---

#### Best Practices

- Use `StepVerifier.withVirtualTime()` for time-sensitive flows
- Keep test methods concise and deterministic
- Test both success and failure paths
- Cover corner cases like empty streams, long delays, and backpressure
- Validate side effects only after `verify()` completes

---

#### Conclusion

Testing reactive applications with **StepVerifier** allows you to assert and control every aspect of a reactive pipeline — from values to timing to errors. It’s a critical tool when working with **Spring WebFlux**, **Project Reactor**, and any non-blocking code in Spring Boot.

Mastering StepVerifier ensures your reactive code is not just scalable and elegant — but also **reliable and testable**.
