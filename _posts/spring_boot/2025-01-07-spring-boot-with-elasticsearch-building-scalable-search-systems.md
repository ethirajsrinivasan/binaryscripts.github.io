---
layout: post
title: Spring Boot with Elasticsearch Building Scalable Search Systems
subtitle: Integrate Spring Boot with Elasticsearch to build robust, high-performance search solutions
categories: Spring Boot
tags: [Java, Spring Boot, Elasticsearch, Search, Big Data, Indexing]
excerpt: Learn how to integrate Elasticsearch with Spring Boot to build scalable search systems. Explore indexing strategies, query optimization, and best practices for high-performance applications.
---



Search is a critical feature in modern applications — from e-commerce and content platforms to enterprise portals and analytics dashboards. Integrating **Spring Boot** with **Elasticsearch** enables developers to build scalable, high-performance search systems using the powerful features of both technologies.

This guide walks through setting up Elasticsearch with Spring Boot, defining indices, performing search queries, and optimizing performance for real-world applications.

---

#### Why Use Elasticsearch with Spring Boot?

Elasticsearch is a distributed, RESTful search engine based on Apache Lucene. Combined with Spring Boot, it offers:

- Full-text and structured search
- Near real-time indexing
- Built-in scalability and fault tolerance
- Seamless integration with REST APIs
- JSON-based query DSL

Spring Boot streamlines Elasticsearch usage by auto-configuring clients and repositories, making development faster and more maintainable.

---

#### Setting Up the Project

Add the following dependencies to your `pom.xml`:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

Specify the Elasticsearch version in `application.yml`:

```yml
spring:
data:
elasticsearch:
client:
reactive:
endpoints: localhost:9200
repositories:
enabled: true
```

Ensure Elasticsearch is running locally or use Docker:

```bash
docker run -d -p 9200:9200 -e "discovery.type=single-node" elasticsearch:8.11.1
```

---

#### Defining an Index with Spring Data Elasticsearch

Create an entity annotated with `@Document`:

```java
@Document(indexName = "products")
public class Product {
@Id
private String id;

    @Field(type = FieldType.Text)
    private String name;

    @Field(type = FieldType.Text)
    private String description;

    @Field(type = FieldType.Double)
    private double price;
}
```

Then define a repository:

```java
public interface ProductRepository extends ElasticsearchRepository<Product, String> {
List<Product> findByNameContaining(String name);
}
```

Spring Data generates the search queries behind the scenes.

---

#### Indexing Documents

Add products to the index via repository:

```java
Product product = new Product("1", "MacBook Pro", "Apple laptop", 1999.99);
productRepository.save(product);
```

Use `saveAll()` to bulk insert documents.

---

#### Performing Search Queries

You can perform structured and full-text queries using repository methods or custom queries.

Repository method:

```java
List<Product> results = productRepository.findByNameContaining("macbook");
```

Custom query using `NativeSearchQueryBuilder`:

```java
SearchHits<Product> hits = elasticsearchOperations.search(
new NativeSearchQueryBuilder()
.withQuery(QueryBuilders.matchQuery("description", "laptop"))
.build(),
Product.class
);
```

Iterate over results:

```java
hits.forEach(hit -> System.out.println(hit.getContent()));
```

---

#### Handling Complex Queries

Elasticsearch supports compound queries using **bool**, **must**, **should**, and **filter** clauses.

```java
BoolQueryBuilder query = QueryBuilders.boolQuery()
.must(QueryBuilders.matchQuery("name", "macbook"))
.filter(QueryBuilders.rangeQuery("price").lte(2500));

SearchHits<Product> results = elasticsearchOperations.search(
new NativeSearchQueryBuilder().withQuery(query).build(),
Product.class
);
```

This enables powerful search experiences like filtered faceted search and relevance ranking.

---

#### Pagination and Sorting

You can paginate and sort results easily:

```java
Pageable pageable = PageRequest.of(0, 10, Sort.by("price").descending());

NativeSearchQuery query = new NativeSearchQueryBuilder()
.withQuery(QueryBuilders.matchAllQuery())
.withPageable(pageable)
.build();
```

This is essential for large datasets or user-facing search interfaces.

---

#### Optimizing Search Performance

Tips for scaling:

- Use **bulk operations** for batch indexing
- Apply **filters** instead of queries where exact matches are used
- Use **keyword** field type for structured data
- Shard wisely — avoid too many small shards
- Avoid frequent mapping updates

Monitoring and tuning are vital for search-heavy apps. Tools like **Elastic HQ** or **Kibana** help visualize index health.

---

#### Securing Elasticsearch APIs

Elasticsearch 8+ includes built-in security. Configure Spring Boot to authenticate with a username/password or API key:

```yml
spring.elasticsearch.uris=https://localhost:9200
spring.elasticsearch.username=elastic
spring.elasticsearch.password=yourpassword
```

Use HTTPS and firewall restrictions to avoid exposing raw Elasticsearch to the internet.

---

#### Best Practices Recap

- Use `@Document` to define index schema
- Keep mappings explicit to avoid type conflicts
- Apply DTOs for API exposure — don’t expose index models directly
- Tune indexing and querying for performance at scale
- Document your search API with OpenAPI/Swagger

---

#### Conclusion

Combining **Spring Boot** with **Elasticsearch** allows you to build rich, performant, and scalable search systems with ease. From e-commerce filters to full-text indexing, Elasticsearch empowers developers to craft intuitive user experiences and robust backend systems.

By mastering the integration patterns and performance optimizations covered here, you’ll be equipped to bring real-time search capabilities to any Java application.
