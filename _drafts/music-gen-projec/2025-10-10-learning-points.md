---
title: 'Music-Gen Project: Key Learning Points'
date: 2025-10-10 13:55:49 +0800
categories: [Java, AI]
tags: [java, artificial-intelligence]
description: TODO
---

## Java language features

### Basic usage of `@RestControllerAdvice` and `@ExceptionHandler` annotations

- `@RestControllerAdvice` is used to handle exceptions globally in Spring Boot applications.
- `@ExceptionHandler` is used to define methods that handle specific exceptions.
- Example usage:
  ```java
  @RestControllerAdvice
  public class GlobalExceptionHandler {
      @ExceptionHandler(Exception.class)
      public ResponseEntity<String> handleException(Exception e) {
        // maybe some other logic in front of this...
          return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(e.getMessage());
      }
  }
  ``` 
- This allows for centralized exception handling and cleaner controller code.
- You can customize the response based on the exception type.
- You can also log exceptions or perform other actions in the handler method.

### How to create a logger using `LoggerFactory`

- `LoggerFactory` is part of the SLF4J (Simple Logging Facade for Java) library.
- It provides a simple and flexible way to create loggers in Java applications.
- Example usage:
  ```java
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  public class MyClass {
      private static final Logger logger = LoggerFactory.getLogger(MyClass.class);
      
      public void myMethod() {
          logger.info("This is an info message");
          logger.error("This is an error message");
      }
  }
  ```
- You can use different logging levels such as `info`, `debug`, `warn`, and `error`.
- This allows for better control over logging output and helps in debugging and monitoring applications.

> **Note: Why we use `MyClass.class` in `getLogger` method?**  
> Using `MyClass.class` as an argument to `getLogger` helps to associate the logger with the specific class. This is useful for identifying the source of log messages, especially in larger applications with many classes.
{: .prompt-info}


### `@RestController` and `@RequestMapping` annotations

- `@RestController` is a specialized version of `@Controller` in Spring Boot that combines `@Controller` and `@ResponseBody`. It is used to create RESTful web services.
- `@RequestMapping` is used to map HTTP requests to handler methods in the controller.
- Example usage:
  ```java
  @RestController
  @RequestMapping("/api")
  public class MyController {

    @Resource
    private MyService myService;

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello, World!";
    }
  }
  ```
- This allows for easy creation of RESTful APIs and handling of different HTTP methods (GET, POST, PUT, DELETE).
- `@Resource` is used for dependency injection, allowing you to inject service beans into the controller.
- You can also use other annotations like `@GetMapping`, `@PostMapping`, etc.


### `@param` annotation in MyBatis

- The `@Param` annotation is used in MyBatis to specify the names of parameters passed to SQL queries.
- It helps in mapping method parameters to SQL query parameters.
- Example usage:
  ```java
  public interface UserMapper {
      Integer updateByParam(@Param("entity") T entity, @Param("query") P query);
  }
  ```
- This allows for better readability and maintainability of SQL queries.
- It is especially useful when you have multiple parameters in a method.
- You can use the parameter names in your SQL queries using `#{parameterName}` syntax.
- Example SQL mapping:
  ```xml
  <update id="updateByParam">
    UPDATE your_table
    SET
        column_a = #{entity.propertyA},
        column_b = #{entity.propertyB}
    WHERE
        some_column = #{query.filterCondition}
  </update>
  ```
- This ensures that the correct values are passed to the SQL queries, reducing the chances of errors.


### using both `@TbaleName`/`@Table` and `Serializable` in entity classes
- `@TableName` (MyBatis Plus) or `@Table` (JPA) annotations are used to specify the database table name that the entity class maps to.
- Implementing `Serializable` in entity classes allows the objects to be converted into a byte stream, which is useful for caching, session storage, or remote communication.
- Example usage:
  ```java
  @Data
  @TableName("tableName")
  public class MusicInfo implements Serializable {

      @Serial
      private static final long serialVersionUID = 1L;

      // other fields and methods...
  }
  ```
- This ensures that the entity class is properly mapped to the database table and can be serialized when needed.
- The `serialVersionUID` is a unique identifier for the class, which helps in version control during serialization and deserialization.
- It is a good practice to include `serialVersionUID` when implementing `Serializable` to avoid potential issues during deserialization if the class structure changes. 
- `@Serial` annotation is used to indicate that the field is related to serialization, but it is not strictly necessary. It is mainly for documentation purposes.
