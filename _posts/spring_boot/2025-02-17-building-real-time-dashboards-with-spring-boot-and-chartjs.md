
---

#### Setting Up Spring Boot with WebSocket

Add the WebSocket dependency in `pom.xml`:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

Configure WebSocket:

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }
}
```

---

#### Sending Real-Time Data from the Server

Simulate metric updates with a scheduled publisher:

```java
@Component
public class MetricPublisher {

    private final SimpMessagingTemplate messagingTemplate;
    private final Random random = new Random();

    public MetricPublisher(SimpMessagingTemplate template) {
        this.messagingTemplate = template;
    }

    @Scheduled(fixedRate = 2000)
    public void publishMetrics() {
        Map<String, Object> metric = Map.of(
            "timestamp", System.currentTimeMillis(),
            "value", 50 + random.nextInt(50)
        );
        messagingTemplate.convertAndSend("/topic/metrics", metric);
    }
}
```

---

#### Frontend with Chart.js and WebSocket

Create `index.html` in `src/main/resources/static/`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Real-Time Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/sockjs-client"></script>
    <script src="https://cdn.jsdelivr.net/npm/stompjs"></script>
</head>
<body>
    <h2>Live Metrics</h2>
    <canvas id="liveChart" width="600" height="400"></canvas>

    <script>
        const ctx = document.getElementById('liveChart').getContext('2d');
        const data = {
            labels: [],
            datasets: [{
                label: 'Metric Value',
                data: [],
                borderColor: 'rgba(75, 192, 192, 1)',
                tension: 0.1
            }]
        };
        const chart = new Chart(ctx, {
            type: 'line',
            data: data,
        });

        const socket = new SockJS('/ws');
        const stompClient = Stomp.over(socket);

        stompClient.connect({}, () => {
            stompClient.subscribe('/topic/metrics', (message) => {
                const metric = JSON.parse(message.body);
                const time = new Date(metric.timestamp).toLocaleTimeString();
                data.labels.push(time);
                data.datasets[0].data.push(metric.value);
                if (data.labels.length > 20) {
                    data.labels.shift();
                    data.datasets[0].data.shift();
                }
                chart.update();
            });
        });
    </script>
</body>
</html>
```

---

#### Optional: Exposing REST Endpoint for Historical Metrics

Provide an endpoint to fetch recent metrics on load:

```java
@RestController
@RequestMapping("/api")
public class DashboardController {

    @GetMapping("/metrics/history")
    public List<Map<String, Object>> getHistory() {
        return IntStream.range(0, 10)
            .mapToObj(i -> Map.of(
                "timestamp", System.currentTimeMillis() - (1000L * i),
                "value", 60 + new Random().nextInt(20)
            )).collect(Collectors.toList());
    }
}
```

You can use this to preload charts before streaming begins.

---

#### Best Practices

- Buffer incoming metrics to avoid memory overflow
- Use `@PreDestroy` or session hooks to clean up subscribers
- Compress WebSocket payloads in production
- Secure WebSocket channels with Spring Security
- Use Kafka or Redis Pub/Sub for scalable real-time sources

---

#### Conclusion

Real-time dashboards provide immediate visibility into system performance, user activity, and business metrics. With **Spring Boot**, **WebSocket**, and **Chart.js**, you can build modern dashboards that are responsive, scalable, and visually rich.

This stack is perfect for admin panels, monitoring tools, and live analytics — giving you the power to build the next generation of real-time web applications.
