---
layout: post
title: Building Flexible Search APIs in Spring Boot with QueryDSL
subtitle: Create dynamic and type-safe search APIs in Spring Boot using the power of QueryDSL
categories: SpringBoot
last_modified_at: "2025-04-03T10:30:00+00:00"
tags: [Spring Boot, QueryDSL, Search API, Java, JPA, Dynamic Queries]
excerpt: Learn how to build flexible and efficient search APIs in Spring Boot using QueryDSL. This guide covers dynamic filters, pagination, and type-safe criteria building with real-world examples.
---
Modern web applications often require **dynamic search functionality** — filtering by optional fields, sorting by user input, and paginating results. Traditional JPA queries or `@Query` annotations in Spring Data can become **verbose and inflexible**.

That’s where **QueryDSL** shines. It allows you to build **type-safe**, **composable**, and **dynamic** queries in Java. In this guide, we’ll walk through how to use **QueryDSL with Spring Boot** to create powerful and reusable search APIs.

---

#### Why Use QueryDSL?

- **Type-safe** query construction using generated classes
- **Readable** and composable fluent APIs
- Perfect for **dynamic filters** and search endpoints
- Seamless integration with Spring Data JPA

QueryDSL generates meta-model classes (like `QUser`) that let you construct queries fluently without using raw SQL or fragile strings.

---

#### Setting Up QueryDSL in Spring Boot

Add the required dependencies in `pom.xml`:

```xml
<dependency>
<groupId>com.querydsl</groupId>
<artifactId>querydsl-jpa</artifactId>
<version>5.0.0</version>
</dependency>

<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-apt</artifactId>
  <version>5.0.0</version>
  <scope>provided</scope>
</dependency>
```

Configure annotation processing (e.g., with Maven):

```xml
<plugin>
<groupId>com.mysema.maven</groupId>
<artifactId>apt-maven-plugin</artifactId>
<version>1.1.3</version>
<executions>
<execution>
<goals><goal>process</goal></goals>
<configuration>
<outputDirectory>target/generated-sources/java</outputDirectory>
<processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
</configuration>
</execution>
</executions>
</plugin>
```

---

#### Create an Entity and Generate Query Types

Example JPA entity:

```java
@Entity
public class Product {
@Id
@GeneratedValue
private Long id;

    private String name;
    private Double price;
    private String category;
}
```

Once compiled, a `QProduct` class will be generated.

---

#### Building a Dynamic Search Query

Use `JPAQueryFactory` to construct dynamic search queries:

```java
public List<Product> searchProducts(String name, String category, Double maxPrice) {
QProduct product = QProduct.product;

    BooleanBuilder builder = new BooleanBuilder();
    if (name != null) builder.and(product.name.containsIgnoreCase(name));
    if (category != null) builder.and(product.category.eq(category));
    if (maxPrice != null) builder.and(product.price.lt(maxPrice));

    return queryFactory
        .selectFrom(product)
        .where(builder)
        .fetch();
}
```

This allows filtering by any combination of parameters.

---

#### Integrating with Spring Data

You can extend `QuerydslPredicateExecutor` in your repository:

```java
public interface ProductRepository extends JpaRepository<Product, Long>,
QuerydslPredicateExecutor<Product> {}
```

Now you can call:

```java
repository.findAll(product.name.containsIgnoreCase("book")
.and(product.category.eq("Education")));
```

---

#### Adding Pagination and Sorting

QueryDSL supports pageable queries:

```java
Page<Product> page = productRepository.findAll(builder, PageRequest.of(0, 10, Sort.by("price").descending()));
```

You can also expose a search endpoint:

```java
@GetMapping("/products/search")
public Page<Product> search(@RequestParam Map<String, String> params, Pageable pageable) {
// build BooleanBuilder dynamically from params
return repository.findAll(builder, pageable);
}
```

---

#### Conclusion

QueryDSL brings **type-safety** and **flexibility** to dynamic query building in Spring Boot. It's ideal for building **search APIs** that need to adapt to user input, support filtering on multiple optional parameters, and scale well with pagination and sorting.

With just a little setup, you can dramatically improve the **maintainability and expressiveness** of your JPA-based search layers.
