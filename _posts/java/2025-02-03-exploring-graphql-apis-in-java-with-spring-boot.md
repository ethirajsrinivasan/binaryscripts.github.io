---
layout: post
title: Exploring GraphQL APIs in Java with Spring Boot
subtitle: Build flexible and efficient GraphQL APIs in Java using Spring Boot and GraphQL Java
categories: Java
tags: [Java, GraphQL, Spring Boot, APIs, Backend, Query Language]
excerpt: Learn how to build and consume GraphQL APIs in Java using Spring Boot. Discover schema design, resolvers, query execution, and best practices for performance and flexibility.
---



Modern applications demand flexible, client-driven APIs — especially for mobile and complex frontend use cases. **GraphQL** has emerged as a powerful alternative to REST, allowing clients to query only the data they need.

In this post, we’ll explore how to build robust **GraphQL APIs in Java using Spring Boot**. We’ll walk through GraphQL schema definition, resolvers, query execution, and best practices — all with real Java code examples.

---

#### What is GraphQL?

**GraphQL** is a query language and runtime developed by Facebook for APIs. Unlike REST, GraphQL allows:
- Fetching multiple resources in a single call
- Strongly-typed query schemas
- Field-level control from the client

Example query:

```graphql
query {
user(id: "123") {
name
email
}
}
```

The server responds with precisely the requested fields — nothing more, nothing less.

---

#### Adding GraphQL to Spring Boot

Use the `spring-boot-starter-graphql` from Spring for an out-of-the-box setup.

Add to your `pom.xml`:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-graphql</artifactId>
</dependency>
```

Also add:

```xml
<dependency>
<groupId>com.graphql-java</groupId>
<artifactId>graphql-java</artifactId>
<version>21.0</version>
</dependency>
```

This sets up the GraphQL engine and auto-configures the `/graphql` endpoint.

---

#### Defining Your GraphQL Schema

Create a file named `schema.graphqls` under `src/main/resources/graphql/`.

```graphql
type Query {
bookById(id: ID!): Book
}

type Book {
id: ID!
title: String
author: String
}
```

This defines a query called `bookById` returning a `Book` type with three fields.

---

#### Writing Resolvers in Java

Create a service and a data resolver:

```java
@Service
public class BookService {
private static final Map<String, Book> BOOKS = Map.of(
"1", new Book("1", "1984", "George Orwell")
);

    public Book getBookById(String id) {
        return BOOKS.get(id);
    }
}
```

Then wire the resolver:

```java
@Component
public class BookResolver implements GraphQlQueryResolver {
private final BookService bookService;

    public BookResolver(BookService bookService) {
        this.bookService = bookService;
    }

    public Book bookById(String id) {
        return bookService.getBookById(id);
    }
}
```

Spring Boot uses GraphQL Java Tools under the hood to wire schema queries to resolver methods automatically.

---

#### Testing Your GraphQL Endpoint

You can test your API using tools like:
- **GraphiQL** (a browser IDE for GraphQL)
- **Postman**
- **Insomnia**

Send a POST request to `/graphql`:

```json
{
"query": "query { bookById(id: \"1\") { title author } }"
}
```

Expected response:

```json
{
"data": {
"bookById": {
"title": "1984",
"author": "George Orwell"
}
}
}
```

---

#### Adding Mutations

Mutations modify data. Extend your schema:

```graphql
type Mutation {
addBook(id: ID!, title: String, author: String): Book
}
```

Add a resolver:

```java
@Component
public class BookMutation implements GraphQlMutationResolver {
private final BookService bookService;

    public BookMutation(BookService bookService) {
        this.bookService = bookService;
    }

    public Book addBook(String id, String title, String author) {
        return bookService.addBook(new Book(id, title, author));
    }
}
```

Now clients can insert data via GraphQL mutation queries.

---

#### Pagination and Filtering

For performance, implement pagination:

```graphql
type Query {
books(page: Int, size: Int): [Book]
}
```

In the resolver:

```java
public List<Book> books(int page, int size) {
return bookRepository.findAll(PageRequest.of(page, size)).getContent();
}
```

Use arguments like `where`, `filter`, or `sort` to expose flexible APIs to consumers.

---

#### Best Practices

- **Avoid over-fetching** by using projections or DataLoaders
- **Validate input** in mutation resolvers
- **Monitor query complexity** to prevent DoS attacks
- **Cache resolved data** to improve performance
- **Version resolvers**, not the schema (keep backward compatibility)

---

#### Conclusion

GraphQL provides a **powerful, flexible API layer** that’s ideal for modern Java applications. With Spring Boot, integration is seamless — from schema to resolver wiring.

By leveraging GraphQL in your backend, you empower frontend teams, reduce over-fetching, and scale APIs elegantly in complex ecosystems.
