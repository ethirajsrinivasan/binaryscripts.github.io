---
layout: post
title: Spring Boot Event Listeners for Real-Time System Monitoring
subtitle: Leverage Spring Boot’s event-driven architecture to build real-time monitoring and auditing features
categories: SpringBoot
tags: [Spring Boot, Event Listener, Monitoring, Observability, Application Events, Real-Time]
excerpt: Learn how to use Spring Boot's event listener system to build real-time monitoring and auditing features. Track system activity, performance, and custom events using a non-invasive, decoupled approach.
---
Monitoring internal activity in real time is crucial for building responsive, reliable applications. While logs and metrics help with external observability, **event-driven monitoring** gives you insight into what’s happening inside your Spring Boot application — as it happens.

Spring Boot provides a powerful **event publishing and listening system**, allowing you to react to application events and custom events in a **decoupled**, **non-blocking**, and **real-time** manner.

In this post, you’ll learn how to use **Spring Boot event listeners** to monitor key system activities, track custom business events, and extend observability beyond metrics and logs.

---

#### What Are Spring Events?

Spring's `ApplicationEventPublisher` allows components to publish events during runtime. These events can then be **asynchronously handled** by `@EventListener` methods.

Out of the box, Spring provides many lifecycle and context-related events:
- `ApplicationStartedEvent`
- `ContextRefreshedEvent`
- `ApplicationReadyEvent`
- `ApplicationFailedEvent`

You can also define **custom events** for domain or system-specific monitoring.

---

#### Listening to Built-in Events

Spring Boot emits several events during startup and runtime. You can listen for them using `@EventListener`.

```java
@Component
public class StartupListener {

    @EventListener
    public void onApplicationReady(ApplicationReadyEvent event) {
        System.out.println("Application is fully started");
    }

    @EventListener
    public void onFailure(ApplicationFailedEvent event) {
        System.err.println("Startup failed: " + event.getException().getMessage());
    }
}
```

These events are perfect for sending alerts or initializing external integrations.

---

#### Creating Custom Events

Define a custom event class by extending `ApplicationEvent` or using a plain object:

```java
public class FileUploadEvent {
private final String filename;
private final Instant timestamp;

    public FileUploadEvent(String filename) {
        this.filename = filename;
        this.timestamp = Instant.now();
    }

    public String getFilename() { return filename; }
    public Instant getTimestamp() { return timestamp; }
}
```

Publish the event using `ApplicationEventPublisher`:

```java
@Autowired
private ApplicationEventPublisher publisher;

public void uploadFile(MultipartFile file) {
// logic to store file
publisher.publishEvent(new FileUploadEvent(file.getOriginalFilename()));
}
```

---

#### Handling Custom Events

To process custom events, create listeners:

```java
@Component
public class FileUploadEventListener {

    @EventListener
    public void handleFileUpload(FileUploadEvent event) {
        System.out.println("File uploaded: " + event.getFilename() + " at " + event.getTimestamp());
        // push to monitoring dashboard, store logs, etc.
    }
}
```

This allows you to track and analyze domain-specific actions in real time.

---

#### Asynchronous Event Handling

By default, event listeners run in the same thread. To make them non-blocking, annotate with `@Async`.

```java
@Async
@EventListener
public void handleAsyncEvent(FileUploadEvent event) {
// long-running task without blocking the main thread
}
```

Enable async support in a config class:

```java
@Configuration
@EnableAsync
public class AsyncConfig { }
```

This is useful for audit logging, notifications, or metrics collection.

---

#### Use Case: Real-Time Monitoring with Events

Use Spring events to monitor:
- User logins
- API call volume
- Failed transactions
- Business KPIs (orders placed, items shipped)

You can persist these events to a log database or stream them to Kafka or Redis for centralized analytics.

Example: Monitoring failed logins

```java
public class LoginFailureEvent {
private final String username;
private final String ipAddress;
private final Instant timestamp;
}
```

Publish the event on login failure and listen to it for audit or alerting.

---

#### Event-Driven Logging and Auditing

Instead of cluttering services with logging logic, emit **domain events** and let separate components handle auditing.

Benefits:
- Separation of concerns
- Easier testing
- Decoupled observability
- Pluggable side-effects

You can also batch or filter events for more efficient persistence or alerting.

---

#### Integrating Events with Monitoring Tools

Use event listeners to:
- Increment custom **Micrometer counters**
- Log structured events for ELK/Graylog
- Push real-time data to WebSocket clients
- Send notifications via Slack, email, etc.

Example: Increment a metric on user registration

```java
@EventListener
public void onUserRegistered(UserRegisteredEvent event) {
meterRegistry.counter("user.registrations").increment();
}
```

---

#### Best Practices

- Use **asynchronous listeners** for non-blocking monitoring
- Keep event classes small and serializable
- Avoid heavy logic in the publisher — move side effects to listeners
- Use event correlation IDs for tracing (combine with MDC)
- Document event contracts like API contracts

---

#### Conclusion

Spring Boot's event system is a powerful yet underused mechanism for building **real-time monitoring** and **auditing solutions**. By leveraging application and custom events, you gain deeper visibility into runtime behavior — all while keeping your codebase clean and modular.

Whether you're tracking file uploads, login attempts, or order processing, event listeners help you build a truly **reactive and observable** Spring Boot application.
