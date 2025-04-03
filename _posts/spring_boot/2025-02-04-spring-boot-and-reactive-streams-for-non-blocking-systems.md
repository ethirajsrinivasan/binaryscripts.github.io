---
layout: post
title: Spring Boot and Reactive Streams for Non-Blocking Systems
subtitle: Build scalable and responsive applications using Spring Boot and reactive programming with Project Reactor
categories: SpringBoot
tags: [Java, Spring Boot, Reactive, WebFlux, Project Reactor, Non-Blocking, Asynchronous]
excerpt: Learn how to build non-blocking, scalable applications using Spring Boot and Reactive Streams. Discover Project Reactor, Spring WebFlux, and the benefits of reactive programming in real-time systems.
---
Modern applications often need to serve thousands or even millions of concurrent users — especially in microservices, messaging systems, or real-time dashboards. Traditional thread-per-request models can’t scale efficiently under such load.

This is where **Reactive Streams** and **non-blocking I/O** come in. With **Spring Boot** and **Spring WebFlux**, Java developers can build **asynchronous, event-driven systems** that handle large volumes of concurrent traffic without exhausting resources.

In this guide, you’ll explore how to build reactive applications with Spring Boot using **Project Reactor**, understand the non-blocking paradigm, and implement performant reactive APIs with WebFlux.

---

#### What are Reactive Streams?

Reactive Streams is a specification that defines a **non-blocking, asynchronous** communication model between components that process streams of data with backpressure.

Key interfaces:
- **Publisher**: Emits items
- **Subscriber**: Consumes items
- **Subscription**: Links them and manages flow
- **Processor**: Both a subscriber and publisher

Spring’s **Project Reactor** provides the core reactive types:
- `Mono<T>`: 0 or 1 result
- `Flux<T>`: 0 to N results

---

#### Why Use Reactive Programming?

Benefits of the reactive approach:
- **Non-blocking I/O** — better scalability with fewer threads
- **Backpressure support** — handle fast producers and slow consumers gracefully
- **Resilience** — easier error propagation and retry logic
- **Responsiveness** — ideal for streaming and real-time use cases

Common use cases:
- Real-time messaging systems
- Chat apps and dashboards
- Streaming APIs
- Microservices with high throughput

---

#### Setting Up Spring Boot with WebFlux

Add the reactive dependencies in your `pom.xml`:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

This enables Spring WebFlux — the reactive alternative to Spring MVC.

---

#### Creating a Reactive REST Controller

Define a controller using `Mono` and `Flux`:

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public Flux<User> getAllUsers() {
        return userService.getAllUsers();
    }

    @GetMapping("/{id}")
    public Mono<ResponseEntity<User>> getUserById(@PathVariable String id) {
        return userService.getUserById(id)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }
}
```

---

#### Writing a Reactive Service

Use `Flux` and `Mono` to build non-blocking pipelines:

```java
@Service
public class UserService {

    private final UserRepository repository;

    public Flux<User> getAllUsers() {
        return repository.findAll();
    }

    public Mono<User> getUserById(String id) {
        return repository.findById(id);
    }
}
```

Repositories can be reactive too, using **R2DBC** or **Mongo Reactive**.

---

#### Reactive MongoDB Example

Add dependency:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
</dependency>
```

Define a reactive repository:

```java
public interface UserRepository extends ReactiveMongoRepository<User, String> {}
```

Now, your entire stack — controller, service, and DB — is non-blocking.

---

#### Backpressure and Flow Control

Backpressure ensures systems don’t crash under high load.

Reactor operators like `limitRate`, `onBackpressureBuffer`, and `timeout` help you manage demand:

```java
Flux.range(1, 1000)
.onBackpressureBuffer(100)
.subscribe(System.out::println);
```

Always test how your reactive components behave under load and apply proper buffering, rate-limiting, and fallbacks.

---

#### Error Handling in Reactive Streams

Reactive error handling uses operators like `onErrorResume`, `onErrorReturn`, and `retry`:

```java
Mono.just("data")
.map(this::process)
.onErrorResume(ex -> Mono.just("fallback"));
```

You can also centralize error handling using `@ControllerAdvice` and WebFlux exception resolvers.

---

#### Streaming and SSE with Flux

Reactive streams shine in server-sent events (SSE) and live updates:

```java
@GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamEvents() {
return Flux.interval(Duration.ofSeconds(1))
.map(seq -> "Event #" + seq);
}
```

This enables dashboards, logs, or metrics to be pushed to clients in real-time.

---

#### Testing Reactive Components

Use `StepVerifier` for unit tests:

```java
StepVerifier.create(service.getUserById("1"))
.expectNextMatches(user -> user.getId().equals("1"))
.verifyComplete();
```

Also integrate reactive test slices with Spring Boot’s `@WebFluxTest`.

---

#### Best Practices

- Use reactive types (`Mono`, `Flux`) end-to-end
- Avoid blocking calls (`Thread.sleep`, JDBC, etc.)
- Apply backpressure handling where needed
- Use reactive database drivers (R2DBC, reactive Mongo)
- Don’t mix blocking and non-blocking code in the same context

---

#### Conclusion

Spring Boot with Reactive Streams provides a modern, scalable way to handle **high-throughput, event-driven applications**. With **Project Reactor**, **Spring WebFlux**, and reactive databases, you can build real-time systems that are resilient and efficient.

By embracing the **non-blocking model**, you reduce thread contention and unlock the full potential of reactive architecture in your Java applications.
