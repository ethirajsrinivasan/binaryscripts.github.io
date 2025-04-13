---
layout: post
title: Real-Time Notifications with Spring Boot and Server-Sent Events
subtitle: Build efficient real-time push notifications in Java using Spring Boot and Server-Sent Events
categories: SpringBoot
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: [Spring Boot, SSE, Server-Sent Events, Real-Time, Notifications, Java, WebSockets]
excerpt: Learn how to implement real-time notifications in Spring Boot applications using Server-Sent Events (SSE). This guide covers streaming data to clients, broadcasting updates, and managing persistent connections.
---
Modern applications are expected to push real-time updates to users — whether it's a chat message, stock price, or system alert. While technologies like **WebSockets** and **polling** exist, **Server-Sent Events (SSE)** offers a simpler, HTTP-based alternative for **one-way streaming from server to client**.

In this post, we’ll explore how to implement **real-time notifications** in a Spring Boot application using SSE. You’ll learn how to stream live events, manage client connections, and broadcast updates with minimal overhead.

---

#### What is Server-Sent Events (SSE)?

SSE is a browser-native technology that allows a server to push updates to the client over a single, long-lived HTTP connection.

Key characteristics:
- Unidirectional: server ➡ client
- Based on standard HTTP/1.1 (no custom protocols)
- Built-in reconnect and retry logic
- Lightweight alternative to WebSockets for real-time feeds

Supported in all major browsers except Internet Explorer.

---

#### When to Use SSE

SSE is ideal for:
- Real-time dashboards
- Live notifications
- System status updates
- Streaming logs
- Chat observers (read-only)

Avoid SSE when you need **two-way communication** — use WebSockets instead.

---

#### Project Setup

Create a Spring Boot project with the following dependencies:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

No additional dependencies are required for SSE support.

---

#### Creating an SSE Controller

Use Spring’s `SseEmitter` to stream events:

```java
@RestController
@RequestMapping("/notifications")
public class NotificationController {

    private final List<SseEmitter> emitters = new CopyOnWriteArrayList<>();

    @GetMapping("/stream")
    public SseEmitter streamNotifications() {
        SseEmitter emitter = new SseEmitter(0L); // no timeout
        emitters.add(emitter);

        emitter.onCompletion(() -> emitters.remove(emitter));
        emitter.onTimeout(() -> emitters.remove(emitter));

        return emitter;
    }

    @PostMapping("/send")
    public ResponseEntity<Void> sendNotification(@RequestBody String message) {
        List<SseEmitter> deadEmitters = new ArrayList<>();
        emitters.forEach(emitter -> {
            try {
                emitter.send(SseEmitter.event().name("notification").data(message));
            } catch (IOException e) {
                deadEmitters.add(emitter);
            }
        });
        emitters.removeAll(deadEmitters);
        return ResponseEntity.ok().build();
    }
}
```

This controller:
- Streams events on `/notifications/stream`
- Broadcasts messages to all connected clients via `/notifications/send`

---

#### Consuming SSE in JavaScript

Clients can subscribe to the stream using native browser APIs:

```html
<script>
  const eventSource = new EventSource("/notifications/stream");

  eventSource.onmessage = function(event) {
    console.log("Message:", event.data);
  };

  eventSource.addEventListener("notification", function(event) {
    alert("New Notification: " + event.data);
  });

  eventSource.onerror = function(err) {
    console.error("Connection error:", err);
  };
</script>
```

This automatically reconnects on disconnection or failure.

---

#### SSE Event Formatting

Spring sends events in the SSE format:

```
event: notification
data: You have a new message!
```

You can customize the event name and add `id`, `retry` headers for client-side handling:

```java
emitter.send(SseEmitter.event()
.id(UUID.randomUUID().toString())
.name("alert")
.data("Disk space low")
.reconnectTime(3000));
```

---

#### Managing Connections

Use `CopyOnWriteArrayList` to handle thread-safe connection management.

Make sure to:
- Remove emitters on timeout or completion
- Avoid memory leaks by pruning dead connections
- Limit the number of concurrent clients if needed

---

#### Deploying SSE Behind a Proxy

When deploying behind NGINX or a load balancer:

- Set high timeouts for long-lived HTTP connections
- Use keep-alive settings
- Disable buffering

Example NGINX config:

```
location /notifications/stream {
proxy_pass http://localhost:8080;
proxy_set_header Connection '';
proxy_http_version 1.1;
chunked_transfer_encoding off;
proxy_buffering off;
proxy_cache off;
proxy_read_timeout 3600s;
}
```

---

#### Alternatives to SSE

| Technology   | Use Case                        | Notes                        |
|--------------|----------------------------------|------------------------------|
| SSE           | Server ➡ Client notifications   | Simple, efficient            |
| WebSocket     | Bi-directional communication    | More complex, full-duplex    |
| Polling       | Compatibility fallback          | High network overhead        |
| gRPC streams  | Microservice communication      | Not browser-native           |

Choose SSE when you want a quick, reliable, and low-latency way to push updates to clients.

---

#### Best Practices

- Set reconnect intervals for resilience
- Use JSON for payloads (`data: {"message":"Hello"}`)
- Consider authentication and authorization on the stream
- Don’t open multiple connections per user unnecessarily
- Monitor emitter count and memory usage

---

#### Conclusion

**Server-Sent Events (SSE)** offers a lightweight and elegant way to implement real-time features in Java applications. With Spring Boot, it's easy to set up persistent connections and stream data efficiently to clients.

If your app needs one-way push notifications, alerts, or live updates — and you want minimal complexity — SSE is a fantastic choice.
