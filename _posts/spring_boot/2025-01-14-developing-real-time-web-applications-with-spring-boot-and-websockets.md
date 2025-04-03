---
layout: post
title: Developing Real-Time Web Applications with Spring Boot and WebSockets
subtitle: Build responsive real-time applications in Java using Spring Boot and WebSocket integration
categories: SpringBoot
tags: [Spring Boot, WebSocket, Real-Time, Java, Messaging, STOMP, SockJS]
excerpt: Learn how to develop real-time, bidirectional communication in your web apps using Spring Boot and WebSockets. From server setup to message broadcasting and client integration, this guide covers it all.
---
Modern web applications increasingly rely on **real-time capabilities** — live chat, notifications, collaborative editing, and real-time dashboards are now expected features. Traditional HTTP is **request-response** and not ideal for pushing updates from server to client.

**WebSockets** provide full-duplex communication between client and server, enabling low-latency, event-driven architectures. This post explains how to use **Spring Boot and WebSockets** to build real-time web applications in Java.

---

#### What Are WebSockets?

WebSocket is a protocol providing:
- **Bidirectional communication** between client and server
- **Persistent TCP connections**
- **Low latency**, ideal for real-time use cases

Unlike HTTP, WebSocket does not require polling. Once a connection is established, data flows in both directions over a single socket.

---

#### Use Cases for WebSockets in Java Applications

- Live chat and customer support interfaces
- Real-time notifications and updates
- Multiplayer game backends
- Collaborative document editing
- IoT and telemetry dashboards

---

#### Setting Up Spring Boot with WebSockets

Add the necessary dependency:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

Create a WebSocket configuration class:

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").withSockJS();
    }
}
```

This enables STOMP messaging over WebSocket and provides fallback via SockJS.

---

#### Creating a Real-Time Message Controller

Create a controller to receive and broadcast messages:

```java
@Controller
public class ChatController {

    @MessageMapping("/chat.send")
    @SendTo("/topic/public")
    public ChatMessage sendMessage(ChatMessage message) {
        return message;
    }
}
```

The `@MessageMapping` maps to client sends, while `@SendTo` broadcasts to subscribers.

Define a message model:

```java
public class ChatMessage {
private String sender;
private String content;
private String type;
// getters and setters
}
```

---

#### Frontend Integration with SockJS and STOMP.js

In your HTML/JavaScript client:

```html
<script src="https://cdn.jsdelivr.net/npm/sockjs-client@1/dist/sockjs.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/stompjs@2.3.3/lib/stomp.min.js"></script>
<script>
    const socket = new SockJS('/ws');
    const stompClient = Stomp.over(socket);

    stompClient.connect({}, function () {
        stompClient.subscribe('/topic/public', function (messageOutput) {
            const message = JSON.parse(messageOutput.body);
            console.log("Received:", message);
        });

        stompClient.send("/app/chat.send", {}, JSON.stringify({
            sender: "user123",
            content: "Hello world!",
            type: "CHAT"
        }));
    });
</script>
```

This establishes a connection and sends messages in real time.

---

#### Broadcasting Notifications

You can broadcast messages from the server (e.g., from a service or scheduler):

```java
@Autowired
private SimpMessagingTemplate messagingTemplate;

public void sendNotification(String data) {
messagingTemplate.convertAndSend("/topic/notifications", data);
}
```

This is useful for system-wide updates like price changes or alerts.

---

#### Securing WebSocket Endpoints

Spring Security supports WebSocket security with a few adjustments:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
http
.authorizeRequests()
.antMatchers("/ws/**").permitAll()
.anyRequest().authenticated()
.and()
.csrf().disable();
}
```

Use message-level authorization:

```java
@Override
public void configureClientInboundChannel(ChannelRegistration registration) {
registration.interceptors(new MyChannelInterceptor());
}
```

You can also include authentication headers in STOMP connections.

---

#### Performance and Scaling

- For small systems, the built-in broker is fine. For production, use RabbitMQ or ActiveMQ as a STOMP broker.
- Limit message size and connection lifetime.
- Use clustering and sticky sessions for WebSocket scalability.
- Monitor connections with `/actuator/websockets` if using Spring Boot Actuator.

---

#### Debugging Tips

- Use browser developer tools to monitor WebSocket frames.
- Check logs for broker errors.
- Use the SockJS fallback only if necessary — prefer native WebSocket.

---

#### Conclusion

WebSockets enable a new class of real-time, responsive web applications. With **Spring Boot and STOMP**, you can implement real-time messaging with minimal overhead and strong integration into the Spring ecosystem.

Whether you're building chat apps, dashboards, or collaborative tools, WebSocket support in Spring Boot gives you everything needed to create **low-latency, scalable**, and **reactive web experiences**.
