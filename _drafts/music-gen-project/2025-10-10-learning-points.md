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

---

### `@Autowired` vs. `@Resource`

`@Autowired` 和 `@Resource` 都是用来进行依赖注入的注解，但它们之间存在一些关键区别，主要体现在来源和默认的注入策略上。

#### 核心区别

1. **来源不同**:
    - `@Autowired` 是 Spring 框架提供的注解。
    - `@Resource` 是 Java 的标准规范 (JSR-250) 中定义的注解。

2. **默认注入方式不同**:
    - `@Autowired` 默认是**按类型 (byType)** 进行注入。
    - `@Resource` 默认是**按名称 (byName)** 进行注入。如果按名称找不到，它会回退到按类型注入。

#### 总结对比

| 特性                   | `@Autowired` (Spring)                         | `@Resource` (Java 标准)                       |
|:-----------------------|:----------------------------------------------|:----------------------------------------------|
| **来源**               | Spring 框架                                   | Java JSR-250 规范                             |
| **默认注入方式**       | **按类型 (byType)**                           | **按名称 (byName)**                           |
| **名称匹配失败后**     | 无后备操作，直接报错（除非只有一个该类型的Bean） | **回退到按类型 (byType)** 注入                |
| **如何解决多Bean歧义** | 使用 `@Qualifier("beanName")`                 | 使用 `name` 属性 `@Resource(name="beanName")` |

#### 示例

假设我们有两个 `MessageService` 的实现：`smsService` 和 `emailService`。

```java
public interface MessageService {
    String getMessage();
}

@Component("smsService")
public class SmsServiceImpl implements MessageService { /* ... */ }

@Component("emailService")
public class EmailServiceImpl implements MessageService { /* ... */ }
```

**使用 `@Autowired`:**

```java
@Autowired
@Qualifier("smsService") // 必须使用 @Qualifier 来指定名称，否则会因找到多个同类型Bean而报错
private MessageService messageService;
```

**使用 `@Resource`:**

```java
// 写法一：通过字段名匹配
@Resource
private MessageService smsService; // 字段名 "smsService" 与 Bean 名匹配，注入成功

// 写法二：通过 name 属性指定
@Resource(name = "emailService")
private MessageService messageService; // 明确指定注入 "emailService"
```

在实际使用中，如果希望代码与 Spring 框架解耦，`@Resource` 是更好的选择。而在纯 Spring 项目中，`@Autowired`（特别是与构造函数注入结合使用时）更为常见。

#### 为什么 `@Autowired` 推荐使用构造函数注入？

将 `@Autowired` 与构造函数结合使用（即构造函数注入）被广泛推荐，因为它能带来更健壮、更可靠、更易于测试的代码。

1. **保证依赖的不可变性 (Immutability)**
   当使用构造函数注入时，你可以将依赖字段声明为 `final`。这意味着一旦对象被创建，它的依赖关系就不能再被更改。这使得你的组件更加稳定和线程安全。

    ```java
    @Service
    public class MyService {
        private final AnotherService anotherService; // 可以声明为 final

        // 从 Spring 4.3 开始，如果类只有一个构造函数，@Autowired 可省略
        public MyService(AnotherService anotherService) {
            this.anotherService = anotherService;
        }
    }
    ```

2. **保证依赖的完整性 (Guaranteed Dependencies)**
   构造函数注入确保了对象在被创建时，其所有必需的依赖都已准备就绪。如果 Spring 无法提供某个依赖，应用会在启动时就失败，这远比在运行时遇到 `NullPointerException` 要好。

3. **提升代码的可测试性 (Improved Testability)**
   在进行单元测试时，你不再需要 Spring 容器。你可以像创建普通 Java 对象一样，使用 `new` 关键字，并手动传入模拟（Mock）的依赖对象。

    ```java
    // 在单元测试中
    AnotherService mockAnotherService = Mockito.mock(AnotherService.class);
    MyService myService = new MyService(mockAnotherService); // 直接创建实例，非常简单
    ```

4. **避免循环依赖 (Avoids Circular Dependencies)**
   当使用构造函数注入时，如果存在循环依赖（A 依赖 B，B 又依赖 A），Spring 在启动时会直接抛出异常，迫使你立即发现并修复这个设计问题。而字段注入可能会掩盖这个问题。

  | 特性           | 构造函数注入 (推荐)  | Setter 注入          | 字段注入 (不推荐) |
  |:---------------|:---------------------|:---------------------|:------------------|
  | **依赖不可变** | ✅ (可使用 `final`)   | ❌                    | ❌                 |
  | **依赖完整性** | ✅ (启动时保证)       | ❌                    | ❌                 |
  | **可测试性**   | ✅ (无需 Spring 容器) | ⚠️ (需要调用 setter) | ❌ (需要反射)      |
  | **循环依赖**   | ✅ (启动时报错)       | ❌ (可能掩盖问题)     | ❌ (可能掩盖问题)  |
