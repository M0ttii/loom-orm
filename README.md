# LOOM-ORM

> **⚠️ University Project Notice**
> This framework was developed as part of a university project. It is in no way production-ready, feature-complete, or optimized for performance. It is intended solely for educational purposes and should not be used in any production environment.

## Table of Contents

1. [What is an ORM?](#what-is-an-orm)
2. [Configure the Database Connection](#1-configure-the-database-connection)
3. [Create Entity Classes](#2-create-entity-classes)
4. [Create a Repository Interface](#3-create-a-repository-interface)
5. [Instantiate the Repository](#4-instantiate-the-repository)
6. [Built-in Repository Methods](#5-built-in-repository-methods)
7. [Dynamic Proxy & Custom Query Methods](#6-dynamic-proxy--custom-query-methods)

---

## What is an ORM?

An **Object-Relational Mapping (ORM)** framework is a programming abstraction that simplifies interaction with relational databases. It allows developers to perform database operations using the constructs of their programming language rather than writing raw SQL. The ORM maps Java objects to database tables and back, abstracting away the low-level data access layer.

This makes development more efficient and produces more maintainable code, as complex SQL statements are expressed as readable method calls on typed objects.

---

## 1. Configure the Database Connection

Set your connection details in the `DatabaseConnection` class to establish a connection to your database.

```java
public class DatabaseConnection {
    private static final String URL = "jdbc:mysql://<HOST>:<PORT>/<DATABASE_NAME>";
    private static final String USER = "<USERNAME>";
    private static final String PASSWORD = "<PASSWORD>";
}
```

---

## 2. Create Entity Classes

An entity class represents a table in the database. The framework uses the following annotations to define the mapping:

| Annotation | Description |
|---|---|
| `@Entity(tableName = "...")` | Marks the class as a database entity and maps it to the specified table. |
| `@Id(name = "...")` | Marks the primary key field. `name` refers to the corresponding database column. |
| `@Column(name = "...")` | Maps a field to a specific database column. |
| `@JoinColumn(name = "...", referencedColumnName = "...")` | Defines a foreign key relationship to another entity. |
| `@CompositeKey(keyColumns = {...})` | Defines a composite primary key spanning multiple columns. |

> **⚠️ Important:** Every entity class **must** include a no-argument constructor. The framework relies on it for reflective instantiation.

### Example: Simple Entity

```java
@Entity(tableName = "user")
public class UserEntity {

    @Id(name = "user_id")
    private int id;

    @Column(name = "username")
    private String username;

    @JoinColumn(name = "address_id", referencedColumnName = "address_id")
    private AddressEntity address;

    // Required no-args constructor
    public UserEntity() {}

    // Getters and setters
}
```

### Example: Composite Primary Key Entity

```java
@Entity(tableName = "order_item")
@CompositeKey(keyColumns = {"order_id", "product_id"})
public class OrderItemEntity {

    @Id(name = "order_id")
    private String orderId;

    @Id(name = "product_id")
    private String productId;

    @Column(name = "quantity")
    private int quantity;

    // Required no-args constructor
    public OrderItemEntity() {}

    // Getters and setters
}
```

---

## 3. Create a Repository Interface

For each entity, define a repository interface that extends `Repository<T, ID>`, where `T` is the entity type and `ID` is the type of the primary key.

```java
public interface UserRepository extends Repository<UserEntity, Integer> {
}
```

The framework will automatically provide implementations of the standard CRUD methods at runtime via Java Dynamic Proxy.

---

## 4. Instantiate the Repository

Repository instances are created through `RepositoryProxy.newInstance()`. No manual implementation is required.

```java
UserRepository userRepository = RepositoryProxy.newInstance(UserRepository.class);
```

---

## 5. Built-in Repository Methods

Every repository automatically provides the following standard operations:

| Method | Description |
|---|---|
| `insert(T entity)` | Inserts a new entity record into the database. |
| `findById(ID id)` | Retrieves an entity by its primary key. |
| `findAll()` | Retrieves all entity records from the database. |
| `update(T entity)` | Updates an existing entity record. |
| `delete(ID id)` | Deletes an entity record by its primary key. |

All `find...()` methods support optional chaining via `.where()` and `.join()` for filtering and joining results, respectively.

### Example: Find by ID

```java
UserEntity user = userRepository.findById(1).findOne();
```

### Example: Insert a New Entity

```java
UserEntity user = new UserEntity();
user.setUsername("john_doe");
userRepository.insert(user);
```

### Example: Find with a WHERE Clause

```java
UserEntity user = userRepository.findAll().where("username", "john_doe");
```

### Example: Find by Composite Primary Key

```java
OrderItemRepository orderItemRepository = RepositoryProxy.newInstance(OrderItemRepository.class);

Map<String, Object> compositeKey = new HashMap<>();
compositeKey.put("order_id", "42");
compositeKey.put("product_id", "7");

OrderItemEntity item = orderItemRepository.findById(compositeKey).findOne();
```

### Example: Execute a Custom SQL Query

```java
String sql = "SELECT * FROM user WHERE username = ?";
UserEntity user = userRepository.executeCustomQuery(sql, "john_doe");
```

---

## 6. Dynamic Proxy & Custom Query Methods

The framework leverages **Java Dynamic Proxy** to interpret method signatures defined in a repository interface and translate them into SQL queries at runtime — without requiring any manual implementation.

To define a custom query method, declare it in the repository interface following the `findBy<ColumnName>` naming convention:

```java
public interface UserRepository extends Repository<UserEntity, Integer> {
    UserEntity findByUsername(String username);
}
```

The proxy will parse the method name, derive the corresponding column and condition, and execute the appropriate SQL query automatically:

```java
UserEntity user = userRepository.findByUsername("john_doe");
```
