---
title: "I Hate ORMs: Hibernate Edition"
date: 2024-07-30
image: "image.png"
description: "Understanding the pitfalls of Hibernate's batch operations and how to optimize them"
categories: ["Java", "Database", "Performance"]
tags: ["Java", "Hibernate", "JPA", "Performance", "Database"]
---

Let me explain why I don't like ORMs. They add a lot of abstraction, especially if you're already working with Java and Spring, which have tons of abstraction layers. Yes, ORMs are useful and powerful, but as the saying goes, with great power comes great responsibility. Here's an example to illustrate my point:

One of the unclear things when you work with JPA and Hibernate is the `saveAll()` method. Many of us might think it does batch inserts by default, but it doesn't. Even when you think you've configured it correctly, it often still doesn't work as expected.

For example, look at this code:

```java
public void batchInsert() {
    var list = List.of(
        new User("nerd1", "n1@nerd.com"),
        new User("nerd2", "n2@nerd.com"),
        new User("nerd3", "n3@nerd.com"),
        new User("nerd4", "n4@nerd.com"),
        new User("nerd5", "n5@nerd.com")
    );
    userRepository.saveAll(list);
}
```

At first look, you might think it performs a batch insert, but in reality, if you enable Hibernate statistics to see what's going on under the hood, you'll find it inserts records one by one:

```properties
spring.jpa.properties.hibernate.generate_statistics=true
```

Here's the log output:

```
796598 nanoseconds spent acquiring 1 JDBC connections;
0 nanoseconds spent releasing 0 JDBC connections;
8030421 nanoseconds spent preparing 5 JDBC statements;
5369411 nanoseconds spent executing 5 JDBC statements;
0 nanoseconds spent executing 0 JDBC batches;
0 nanoseconds spent performing 0 L2C puts;
0 nanoseconds spent performing 0 L2C hits;
0 nanoseconds spent performing 0 L2C misses;
3607503 nanoseconds spent executing 1 flushes (flushing a total of 5 entities and 0 collections);
0 nanoseconds spent executing 0 pre-partial-flushes;
0 nanoseconds spent executing 0 partial-flushes (flushing a total of 0 entities and 0 collections)
```

You can see `executing 5 JDBC statements;` and `executing 0 JDBC batches;`

To enable batch insert in Hibernate, you need to add these configs to `application.properties`:

```properties
spring.jpa.properties.hibernate.jdbc.batch_size=10
spring.jpa.properties.hibernate.order_inserts=true
```

And when you run the code again, you will still find the problem LOL (:

```
678067 nanoseconds spent acquiring 1 JDBC connections;
0 nanoseconds spent releasing 0 JDBC connections;
8262300 nanoseconds spent preparing 5 JDBC statements;
4660724 nanoseconds spent executing 5 JDBC statements;
0 nanoseconds spent executing 0 JDBC batches;
0 nanoseconds spent performing 0 L2C puts;
0 nanoseconds spent performing 0 L2C hits;
0 nanoseconds spent performing 0 L2C misses;
3438648 nanoseconds spent executing 1 flushes (flushing a total of 5 entities and 0 collections);
0 nanoseconds spent executing 0 pre-partial-flushes;
0 nanoseconds spent executing 0 partial-flushes (flushing a total of 0 entities and 0 collections)
```

If you set the Hibernate log level to DEBUG, you'll see something interesting:

```properties
logging.level.org.hibernate=DEBUG
```

You will see `Executing identity-insert immediately` repeated five times, which is the size of our list:

```
.... : Executing identity-insert immediately
.... : insert into users (email,username) values (?,?) returning id
```

That's weird, right? This happens because when you have an entity with an ID that auto-increments using `GenerationType.IDENTITY`, each insert statement must be executed immediately to obtain the generated key, making it impossible to batch multiple inserts into a single statement.

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String email;
}
```

To fix that, you have to change the GeneratedValue strategy to SEQUENCE. This allows Hibernate to use its own sequence generator, which is compatible with batch inserts. It requires additional select statements to get the next value from a database sequence, but this usually has no significant performance impact.

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;
    private String username;
    private String email;
}
```

Let's try this and see the logs:

```
1169049 nanoseconds spent acquiring 1 JDBC connections;
0 nanoseconds spent releasing 0 JDBC connections;
13152005 nanoseconds spent preparing 3 JDBC statements;
5535759 nanoseconds spent executing 2 JDBC statements;
2062795 nanoseconds spent executing 1 JDBC batches;
0 nanoseconds spent performing 0 L2C puts;
0 nanoseconds spent performing 0 L2C hits;
0 nanoseconds spent performing 0 L2C misses;
19371806 nanoseconds spent executing 1 flushes (flushing a total of 5 entities and 0 collections);
0 nanoseconds spent executing 0 pre-partial-flushes;
0 nanoseconds spent executing 0 partial-flushes (flushing a total of 0 entities and 0 collections)
```

Success! It works. You can see executing 2 JDBC statements: one to select the next ID (which happens when you use `GenerationType.SEQUENCE`) and the other for the batch insert (1 JDBC batches).

From 5 to 2 — imagine if you were inserting 100 records. This would make a big difference and have a huge impact on performance.

## Conclusion

While JPA and Hibernate provide powerful abstractions, understanding their underlying mechanisms is crucial for optimizing performance. By properly configuring batch inserts and being aware of potential pitfalls, you can significantly enhance the efficiency of your database operations, especially when dealing with large datasets.

Remember, the key to effective use of ORMs lies in balancing their convenience with a deep understanding of their behavior and limitations. Always test and verify your assumptions to ensure your application performs as expected.