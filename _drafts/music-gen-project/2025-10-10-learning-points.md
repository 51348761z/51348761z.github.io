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
- This ensures that the entity class is properly mapped to the database table and can be serialized when needed.
- The `serialVersionUID` is a unique identifier for the class, which helps in version control during serialization and deserialization.
- It is a good practice to include `serialVersionUID` when implementing `Serializable` to avoid potential issues during deserialization if the class structure changes.
- `@Serial` annotation is used to indicate that the field is related to serialization, but it is not strictly necessary. It is mainly for documentation purposes.

### `@Configuration`, `@Bean` and `@Component` annotations in Spring Boot

- `@Configuration` is used to indicate that a class contains bean definitions for the Spring application context.
- `@Bean` is used to define a bean that will be managed by the Spring container.
- `@Component` is a generic stereotype annotation that indicates that a class is a Spring-managed component.
- Example usage:

  ```java
  @Configuration
  public class AppConfig {

      @Bean
      public MyService myService() {
          return new MyServiceImpl();
      }
  }
  ```

- This allows for easy configuration and management of beans in a Spring Boot application.
- You can use `@Component` to annotate classes that should be automatically detected and registered as beans by Spring's component scanning.
- Example usage of `@Component`:

  ```java
  @Component
  public class MyComponent {
      // component logic...
  }
  ```

- This helps in reducing boilerplate code and promotes a cleaner architecture by separating configuration from business logic.

#### Differences between `@Configuration`, `@Bean`, and `@Component`

| Annotation       | Purpose                                          | Usage Context                                   |
|------------------|--------------------------------------------------|-------------------------------------------------|
| `@Configuration` | Indicates a class that contains bean definitions | Used on classes that define beans               |
| `@Bean`          | Defines a bean that will be managed by Spring    | Used on methods within `@Configuration` classes |
| `@Component`     | Marks a class as a Spring-managed component      | Used on classes to enable component scanning    |

`@Component` 和 @`Configuration` 都用于将类声明为 Spring IoC 容器中的 Bean，但它们的使用场景有所不同。

以下是选择使用哪一个的指导原则：

1. 使用 `@Component`（或其特化注解如 `@Service`, `@Repository`, `@Controller`）：  
   当你希望 Spring 自动扫描并注册你自己编写的类作为 Bean 时，使用此注解。
   这适用于应用中的业务逻辑组件、数据访问组件或控制器。
   你直接在需要被 Spring 管理的类上添加此注解。
   简单来说： 如果这个类是你自己写的，并且你希望 Spring 为你创建和管理它的实例，就用 `@Component`。
2. 使用 `@Configuration`：  
   当你需要定义不属于你项目代码的 Bean（例如，来自第三方库的类）时，或者当 Bean 的创建过程比较复杂，需要特定逻辑时，使用此注解。
   `@Configuration` 注解的类本身也是一个组件，但它的主要目的是作为 Bean 定义的来源。
   在 `@Configuration` 类中，你会使用 `@Bean` 注解来修饰方法，这些方法负责创建和配置 Bean 实例。
   简单来说： 如果你想将一个外部库的类或者需要复杂初始化的对像注册为 Bean，就在一个带有 `@Configuration` 的类中，创建一个返回该对象实例并用 `@Bean` 注解的方法。

| Use Case                               | Annotation(s)                | Purpose                                                              |
|----------------------------------------|------------------------------|----------------------------------------------------------------------|
| For your own business components       | `@Component`                 | Marks a class for Spring to auto-scan and create an instance.        |
| For 3rd-party or complex beans         | `@Configuration` and `@Bean` | Acts as a factory to define, create, and configure beans via methods.|

### `@RequestBody` and `@RequestParam` annotations
