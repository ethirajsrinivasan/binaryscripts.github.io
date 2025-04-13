---
layout: post
title: Spring Boot with MongoDB for NoSQL Applications
subtitle: Build flexible and scalable NoSQL applications with Spring Boot and MongoDB
categories: SpringBoot
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: [Java, Spring Boot, MongoDB, NoSQL, Database, Reactive, JSON, Document Store]
excerpt: Learn how to integrate Spring Boot with MongoDB to develop powerful NoSQL applications. Explore document modeling, repository usage, query techniques, and best practices for real-world deployments.
---
Traditional relational databases aren't always the best fit for modern, dynamic applications. **NoSQL databases** like **MongoDB** offer schema-less flexibility, rich JSON document structures, and horizontal scalability — making them ideal for microservices, real-time analytics, and cloud-native applications.

In this post, you'll learn how to integrate **Spring Boot with MongoDB**, model data as documents, perform CRUD operations, write custom queries, and apply best practices to build robust NoSQL applications.

---

#### Why Use MongoDB with Spring Boot?

MongoDB is a **document-oriented database** that stores data in flexible, JSON-like BSON format. Key benefits include:

- **Schema flexibility** – Easily evolve data structures
- **Rich querying** – Indexing, filtering, and aggregation
- **Horizontal scalability** – Sharding support
- **Built-in replication** – High availability
- Native integration with Spring via Spring Data MongoDB

---

#### Add Spring Data MongoDB Dependency

Add the starter to your `pom.xml`:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

This brings in Spring Data MongoDB, Mongo Java Driver, and Spring Boot auto-configuration.

---

#### Configure MongoDB Connection

Set the connection details in `application.yml`:

```yml
spring:
data:
mongodb:
uri: mongodb://localhost:27017/mydb
```

Optionally, separate authentication credentials:

```yml
spring:
data:
mongodb:
host: localhost
port: 27017
username: user
password: pass
database: mydb
```

---

#### Define a MongoDB Document

Use `@Document` to mark a class as a MongoDB document:

```java
@Document(collection = "users")
public class User {

    @Id
    private String id;

    private String name;

    private String email;

    private LocalDateTime createdAt = LocalDateTime.now();

    // Getters and setters
}
```

MongoDB automatically creates the `_id` field and stores the document in BSON format.

---

#### Create a Repository Interface

Use `MongoRepository` or `ReactiveMongoRepository`:

```java
public interface UserRepository extends MongoRepository<User, String> {
List<User> findByName(String name);
Optional<User> findByEmail(String email);
}
```

Spring Data automatically implements this interface with dynamic query derivation based on method names.

---

#### REST Controller Example

Expose MongoDB-backed data via a Spring Boot REST API:

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserRepository repository;

    @PostMapping
    public ResponseEntity<User> create(@RequestBody User user) {
        return ResponseEntity.ok(repository.save(user));
    }

    @GetMapping
    public List<User> list() {
        return repository.findAll();
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> get(@PathVariable String id) {
        return repository.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
}
```

---

#### Custom Queries with @Query Annotation

Use MongoDB's powerful query language with annotations:

```java
@Query("{ 'email': ?0 }")
Optional<User> findByEmailCustom(String email);
```

You can also use aggregation pipelines for advanced use cases like grouping, projections, and joins.

---

#### Pagination and Sorting

Spring Data MongoDB supports paging:

```java
@GetMapping("/page")
public Page<User> paginate(Pageable pageable) {
return repository.findAll(pageable);
}
```

Request example:

```http
GET /users/page?page=0&size=5&sort=name,asc
```

---

#### Using ReactiveMongoRepository (Optional)

For high-throughput reactive applications, use the reactive stack:

```java
public interface ReactiveUserRepository extends ReactiveMongoRepository<User, String> {
Flux<User> findByName(String name);
}
```

Enable reactive support:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
</dependency>
```

---

#### Indexing and Performance

Index fields you filter or sort on frequently:

```java
@CompoundIndex(name = "email_idx", def = "{'email': 1}", unique = true)
public class User { ... }
```

Or manually in MongoDB shell:

```js
db.users.createIndex({ email: 1 }, { unique: true });
```

---

#### Best Practices

- Use **UUIDs** or MongoDB's ObjectId for primary keys
- Normalize for consistency, denormalize for performance
- Design documents based on query patterns
- Monitor using MongoDB Atlas or Prometheus exporters
- Validate schema using DTOs and `@Valid`

---

#### Conclusion

Spring Boot + MongoDB is a powerful combination for building modern, scalable NoSQL applications. With flexible schemas, robust query capabilities, and Spring’s seamless integration, you can build and scale services that are fast, fault-tolerant, and cloud-ready.

Whether you're developing microservices, IoT platforms, or user analytics tools, MongoDB offers the flexibility and performance needed in today’s dynamic data landscape.
