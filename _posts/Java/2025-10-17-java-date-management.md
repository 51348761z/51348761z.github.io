---
title: "Java 日期时间类型"
date: 2025-10-17 15:40:49 +0800
categories: ["Java", "Date Management"]
tags: ["Java", "Date", "Time", "LocalDateTime", "Jackson", "JsonFormat"]
draft: true
---

在 Java 开发中，将数据库的 `DATETIME` 或 `TIMESTAMP` 类型字段映射到实体类时，通常会选用 `java.util.Date` 或 `java.time.LocalDateTime`。尽管两者都能表示时间，但它们在设计、功能和最佳实践上存在显著差异。

**核心建议：在新项目中，应优先使用 `LocalDateTime` 及其所属的 `java.time` API。**

```java
// 旧版 API
private Date createTime;
private Date updateTime;

// 推荐的 Java 8+ API
private LocalDateTime createTime;
private LocalDateTime updateTime;
```

## 一、`Date` 与 `LocalDateTime` 的核心区别

### 1. `java.util.Date` (旧版 API)

- **本质**：`Date` 类型本质上是一个时间戳，它内部存储的是自 1970 年 1 月 1 日 00:00:00 GMT 以来的**毫秒数**。
- **可变性**：`Date` 对象是**可变的**。这意味着你可以通过 `setTime()` 等方法修改一个已存在的 `Date` 对象的值，这在多线程环境下可能引发问题。
- **时区**：`Date` 对象本身不包含任何时区信息，它代表的是一个精确的、全球唯一的时刻（UTC 时间）。但在进行字符串转换或显示时，它会依赖 JVM 的默认时区，这常常导致混淆和错误。
- **API 设计**：其 API 设计（如年份偏移、月份从 0 开始计数）被认为是过时且不直观的。

### 2. `java.time.LocalDateTime` (Java 8+ 新版 API)

- **本质**：`LocalDateTime` 代表一个**不带时区**的“日期+时间”描述。例如，“2025年10月26日上午10点”。它本身无法对应到一个全球唯一的精确时刻，除非结合时区信息。
- **不可变性**：`LocalDateTime` 对象是**不可变的**。任何修改操作（如 `plusDays()`, `withHour()`）都会返回一个全新的实例，这使其天然线程安全。
- **时区**：它明确地将日期时间与时区解耦。如果需要处理时区，应使用 `ZonedDateTime` 或 `OffsetDateTime`。
- **API 设计**：提供了丰富且语义清晰的 API，如 `of()`, `parse()`, `plus()`, `minus()` 等，使日期时间操作变得简单直观。

## 二、JSON 输出格式化

在 Spring Boot 项目中，当通过 REST API 返回包含日期时间字段的 JSON 时，我们通常需要将其格式化为统一的字符串格式。这通常借助 Jackson 库实现。

首先，请确保项目中包含 `jackson-databind` 依赖（`spring-boot-starter-web` 已默认引入）。

````xml
<!-- jackson-databind dependency -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.3</version> <!-- 请使用项目适配的版本 -->
</dependency>
`````

下面补充两种常见且实用的做法：全局配置（推荐）和字段级注解（局部覆盖）。

### 方法一：全局配置（推荐）

在 `application.properties` 或 `application.yml` 中进行全局配置，可以为整个应用设定统一的日期时间格式，便于维护和保证前后端一致性。这是推荐的做法。

application.properties 示例：

```properties
# 全局设置日期时间格式
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
# 设置时区，确保时间转换的准确性
spring.jackson.time-zone=GMT+8
```

application.yml 示例：

```yaml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
```

> - `spring.jackson.date-format`：控制 Jackson 对 `java.util.Date`、`java.time` 等时间类型的序列化/反序列化格式。
> - `spring.jackson.time-zone`：确保在序列化/反序列化时使用统一时区（防止因 JVM 默认时区差异导致的偏移）。
{: .prompt-info }

### 方法二：字段级注解（局部覆盖）

当某个字段需要与全局配置不同的显示格式时，可以在实体类字段上使用 `@JsonFormat` 注解进行局部覆盖，灵活但会使配置分散。

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import java.time.LocalDateTime;

public class YourEntity {

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private LocalDateTime startTime;
 
    @JsonFormat(pattern = "yyyy/MM/dd", timezone = "GMT+8") // 使用不同的格式
    private LocalDateTime eventDate;
}
```

- `pattern`：指定日期时间的输出/解析格式。
- `timezone`：指定用于格式化的时区（推荐显式指定以避免时区陷阱）。

### 小结

- 若项目中对时间格式有统一要求，优先使用全局配置（`spring.jackson.*`）。
- 在少数需要特殊格式的字段上使用 `@JsonFormat` 做局部覆盖。
- 如果需要更复杂的控制（例如不同 API 返回不同格式、或自
