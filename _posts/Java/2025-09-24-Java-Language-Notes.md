---
title: Java Language Notes
date: 2025-09-24 14:00:00 +0800
categories: [Java]
tags: [java, springboot, jpa, annotation]
description: A collection of notes and key points related to the Java programming language and some frameworks.
---

## Use `Optional.map()`

Java 中 `Optional.map()` 的基本用法总结
`Optional.map()` 是 Java 8+ 中 `Optional` 类的方法，用于函数式编程。它对 `Optional` 的值应用一个映射函数（Function），返回一个新的 `Optional`。如果原 `Optional` 为空，则直接返回空的 `Optional`。

```java
public <U> Optional<U> map(Function<? super T, ? extends U> mapper)
```
**基本操作**
- 有值时：应用映射函数，返回包含转换后值的 Optional。
- 为空时：不应用函数，返回 `Optional.empty()`。
exampl:
```java
import java.util.Optional;

```java
// 示例 1: 有值的情况
Optional<String> opt = Optional.of("hello");
Optional<Integer> length = opt.map(String::length); // Optional[5]

// 示例 2: 空值的情况
Optional<String> emptyOpt = Optional.empty();
Optional<Integer> emptyLength = emptyOpt.map(String::length); // Optional.empty

// 示例 3: 链式调用
Optional<Student> studentOpt = studentService.findOneStudent(id);
ResponseEntity<Student> response = studentOpt
    .map(ResponseEntity::ok)  // 如果有值，转换为 ResponseEntity.ok(student)
    .orElse(ResponseEntity.notFound().build());  // 如果为空，返回 404
```

**Key feature**:
- 避免空指针：比手动检查 `isPresent()` 更简洁。
- 链式操作：常与 `orElse()`、`orElseGet()` 等结合使用。
- 类型转换：映射函数可以改变类型（如 `Student` → `ResponseEntity<Student>`）。
- 性能：映射只在有值时执行，无额外开销。

## What is `@Autowired`

`@Autowired` 是 Spring 框架中的一个注解，用于实现依赖注入 (Dependency Injection, DI)。它可以自动将 Spring 容器中的 Bean 注入到类的字段、构造函数或方法中，从而简化了对象的创建和管理。

**基本用法**

- 字段注入：直接在字段上使用 `@Autowired`。
```java 
@Autowired
private StudentService studentService;
```

- 构造函数注入：在构造函数上使用 `@Autowired`（推荐）。
```java
@Autowired
public StudentController(StudentService studentService) {
    this.studentService = studentService;
}
```

- 方法注入：在 setter 方法上使用 `@Autowired`。
```java
@Autowired
public void setStudentService(StudentService studentService) {
    this.studentService = studentService;
}
```

**工作原理**
- Spring 容器启动时，会扫描带有 `@Component`、`@Service`、`@Repository` 等注解的类，并将其实例化为 Bean。
- 当遇到 `@Autowired` 注解时，Spring 会查找匹配类型的 Bean，并将其注入。如果找到多个匹配的 Bean，可以使用 `@Qualifier` 指定具体的 Bean 名称。

**注意事项**
- 默认情况下，`@Autowired` 是按类型注入的。如果没有找到匹配的 Bean，会抛出异常。可以通过设置 `required=false` 来避免异常。
- 依赖循环：如果两个 Bean 相互依赖，可能会导致循环依赖问题。可以通过使用 `@Lazy` 注解来解决。
- 作用域：`@Autowired` 可以与不同作用域的 Bean 一起使用，如单例、原型等。

## Java Stream API 在数据转换中的应用
Java Stream API 是 Java 8 引入的功能式编程工具，用于处理集合数据。它允许以声明式方式进行数据转换、过滤和聚合，提高代码可读性和效率。在 Spring Boot 项目中，Stream 常用于将实体转换为 DTO，或进行数据处理。

以下是 Stream API 的核心组件及用法，以 `findActiveStudents()` 方法为例：

```java
@Override
public List<StudentDTO> findActiveStudents() {
    return studentRepository.findStudentsByActiveTrue().stream()
            .map(this::convertToDto)
            .collect(Collectors.toList());
}
```

### 11.1 Stream
- **说明**：`stream()` 方法将一个集合（如 `List<Student>`）转换为 `Stream<Student>` 对象。Stream 是一种数据管道，支持链式操作，但不修改原集合。
- **用法**：调用集合的 `stream()` 方法启动管道。Stream 是惰性的，只有在终端操作（如 `collect`）时才会执行。
- **示例**：`studentRepository.findStudentsByActiveTrue().stream()` 创建一个学生实体的 Stream。

### 11.2 map
- **说明**：`map()` 是中间操作，用于对 Stream 中的每个元素应用转换函数，返回新的 Stream。这里 `this::convertToDto` 是方法引用，将 `Student` 实体转换为 `StudentDTO`。
- **用法**：接受一个 `Function<T, R>` 函数式接口。适用于一对一转换，如实体到 DTO。
- **示例**：`.map(this::convertToDto)` 将每个 `Student` 映射为 `StudentDTO`。

### 11.3 collect
- **说明**：`collect()` 是终端操作，将 Stream 的元素收集成一个结果容器（如 List、Set 或 Map）。它是 Stream 管道的终点，触发所有中间操作的执行。
- **用法**：接受一个 `Collector` 对象，定义如何累积元素。常用 `Collectors` 工具类提供预定义收集器。

### 11.4 Collectors
- **说明**：`Collectors` 是工具类，提供静态方法创建 `Collector` 实例。`toList()` 创建一个收集器，将元素收集到 `ArrayList` 中。
- **用法**：`Collectors.toList()` 返回 `List<T>`；其他如 `toSet()`、`toMap()` 等。适用于聚合结果。
- **示例**：`.collect(Collectors.toList())` 将 `StudentDTO` Stream 收集为 `List<StudentDTO>`。

**优势**：Stream API 支持并行处理（用 `parallelStream()`）、链式调用和函数式风格，减少样板代码。但对于简单循环，传统 for 循环可能更高效。注意 Stream 不可重用，一旦消费（如 collect），需重新创建。

