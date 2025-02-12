## Purpose - Testing GitHub Actions
# Change Line: XYDKJFKDGG


## Introduction to Micronaut
This is a sample project in micronaut that draws the basic idea of:
- CRUD operation
- Connecting to a MySql database
- Database migration using Flyway
- Global exception handler
- JPA and JDBC

Micronaut is heavily dependent on annotaion processor. So, at the very first, the annotation processor should be enabled from the settings of your IDE.

Below are some important points I have came across while exploring micronaut.

## Using application.yaml instead of application.properties
To enable the support of `application.yaml` the following runtime only dependency is required. Until the below dependency is added, the micronaut application will not load configuration from the `application.yaml` file. Instead, it will look for only the `application.properties` file.

*Dependency:*
```groovy
runtimeOnly("org.yaml:snakeyaml")
```
After adding this dependency, you can replace your `appllication.properties` file with `application.yaml` file.

## Adding lombok support
As micronaut is heavily dependent on annotation processing, while adding lombok, we should also add the annotation processor dependency to make it work correctly

*Dependenies:*
```groovy
compileOnly("org.projectlombok:lombok")
annotationProcessor("org.projectlombok:lombok")
```

## Serialization-Deserialization
The serde processor is used for serialization and deserialization in micronaut. To use the serde processor, below dependencies should be added.

```groovy
implementation("io.micronaut.serde:micronaut-serde-jackson")
annotationProcessor("io.micronaut.serde:micronaut-serde-processor")
```
Once the dependencies are added, we can use the `@Serde` annotation to enable serialization and deserialization for classes. It will use jackson serialization and deserialization.

## Adding flyway support for db migration
Flyway is a great db migration tool that itself manages the database versions. As in production, we should not rely upon the JPA to create and manage the production db, it is a reliable tool for the same.

To add the support of flyway in our micronaut application, we should add the below dependencies first to get started.

*Dependencies:*
```groovy
implementation("io.micronaut.flyway:micronaut-flyway")
implementation("org.flywaydb:flyway-mysql")
```

After adding the dependencies, we need to add a sql script that will execute when the app starts for the first time. Also, it will verify the db migration everytime the application starts up. We need to add the script to the `reousrces/db/migration` directory and should name the file `V1__any_name.sql`. Here the version is important as the flyway executes the scripts according to the version number.

A sample script is given below which creates a table named books:
```sql
CREATE TABLE IF NOT EXISTS books
(
    id         BIGINT         NOT NULL AUTO_INCREMENT UNIQUE PRIMARY KEY,
    name       VARCHAR(255)   NOT NULL,
    author     VARCHAR(255)   NOT NULL,
    price      DECIMAL(10, 2) NOT NULL,
    total_page INT            NOT NULL
);
```

Also, you need to add the below configuration in your `application.yaml` to enable the flyway support:
```
flyway:
  datasources:
    default:
      enabled: true
```


## Micronaut-data and database
There are basically two approaches to connect to a database in micronaut. One is using `micronaut-data-jdbc` and another is `micronaut-data-jpa`.
You need to add the below configuration to your `application.yaml` file and then choose one of the below ways to connect to a database. For simplicity, I am using `MySql` database throughout this project

```
datasources:
  default:
    url: datasource_url:port/db_name
    username: your_username
    password: your_password
    driver-class-name: com.mysql.cj.jdbc.Driver
```

---

### JDBC
By following the below steps, you will be able to use the JPA in your micronaut application.

*First, you need to add the below dependencies:*
```groovy
implementation("io.micronaut.data:micronaut-data-jdbc") #for jdbc
implementation("io.micronaut.sql:micronaut-jdbc-hikari")
runtimeOnly("mysql:mysql-connector-java")
```
*The model class will be look like this:*
```java
package com.piyal.model;

import io.micronaut.data.annotation.MappedEntity;
import io.micronaut.data.annotation.Id;
import io.micronaut.serde.annotation.Serdeable;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@MappedEntity(value = "books") // for mapping this model to the db table; value is the table name
@Serdeable // for serialization and deserialization
@AllArgsConstructor
@NoArgsConstructor
@Data
public class Book {
    @Id
    private Long id;
    private String name;
    private String author;
    private Double price;
    private Integer totalPage;
}
```

The repository interface should be annotated with `@JdbcRepository(dialect = Dialect.MYSQL)` and should extend `CrudRepository<Entity, IdType>`

*The repository will look like this:*
```java
package com.piyal.repository;

import com.piyal.model.Book;
import io.micronaut.data.jdbc.annotation.JdbcRepository;
import io.micronaut.data.model.query.builder.sql.Dialect;
import io.micronaut.data.repository.CrudRepository;

@JdbcRepository(dialect = Dialect.MYSQL)
public interface BookRepository extends CrudRepository<Book, Long> {
}

```

### JPA
Using JPA with micronaut can be tricky. By following the below steps, you will be able to use the JPA in your micronaut application.

*First, you need to add the below dependencies:*
```groovy
implementation("io.micronaut.data:micronaut-data-hibernate-jpa") #for the jpa
implementation("io.micronaut.sql:micronaut-jdbc-hikari")
runtimeOnly("mysql:mysql-connector-java")
```
After that, you need to add the below configuration to your `application.yaml` file:
```
jpa:
  default:
    properties:
      hibernate:
        hbm2ddl:
          auto: none #this will not create the table using jpa as we are already maintaining our db using flyway
        show_sql: true
```

*The model class will look like this:*
```java
package com.piyal.model;

import io.micronaut.serde.annotation.Serdeable;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Table(name = "books")
@Serdeable
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Book {
    @Id
    private Long id;
    private String name;
    private String author;
    private Double price;
    private Integer totalPage;
}
```

And now the repository interface can be annotated with `@Repository` and can extends `JpaRepository<Entity, IdType>`

*The repository will look like this:*
```java
package com.piyal.repository;

import com.piyal.model.Book;
import io.micronaut.data.annotation.Repository;
import io.micronaut.data.jpa.repository.JpaRepository;

@Repository
public interface BookRepository extends JpaRepository<Book, Long> {
}

```