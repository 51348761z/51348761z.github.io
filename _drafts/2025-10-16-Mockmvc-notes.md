---
title: "MockMVC & Java Testing Notes"
date: 2025-10-16T04:46:33+08:00
categories: ["Java", "Test"]
tags: ["MockMVC", "Spring Boot", "Unit Testing"]
description: "Notes on using MockMVC for testing Spring Boot applications."
---

## 什么是 MockMVC？

`MockMvc` 是 Spring Test 框架的一部分，专门用于测试 Spring MVC 控制器（Controller）。它提供了一种模拟 HTTP 请求并对控制器进行测试的方式，而**无需启动一个完整的 HTTP 服务器**。

通过 `MockMvc`，你可以在一个模拟的 Servlet 环境中测试你的 Controller，就像一个真实的客户端在发送请求一样。它会处理请求映射、数据绑定、类型转换、验证等所有 Spring MVC 的标准流程，但这一切都发生在测试环境中，速度快且不依赖于网络。

## 简单用法和核心组件

使用 `MockMvc` 通常遵循“三步走”的模式：**执行请求 -> 验证结果 -> （可选）打印结果**。

### 1. 准备测试环境

首先，你需要在测试类上使用 `@SpringBootTest` 和 `@AutoConfigureMockMvc` 注解，并注入 `MockMvc` 实例。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;

@SpringBootTest
@AutoConfigureMockMvc
public class MyControllerTest {

    @Autowired
    private MockMvc mockMvc;

    // ... tests go here
}
```

- `@SpringBootTest`: 加载完整的 Spring 应用上下文。
- `@AutoConfigureMockMvc`: 自动配置 `MockMvc` 实例。

### 2. 构建和执行请求

使用 `mockMvc.perform()` 方法来执行一个请求。`MockMvcRequestBuilders` 类提供了静态方法来构建各种类型的 HTTP 请求（GET, POST, PUT, DELETE 等）。

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@Test
public void testGetUserById() throws Exception {
    mockMvc.perform(get("/api/users/1")); // 执行一个 GET 请求
}
```

### 3. 验证响应结果

`perform()` 方法返回一个 `ResultActions` 对象，你可以链式调用 `.andExpect()` 方法来添加多个断言。`MockMvcResultMatchers` 提供了丰富的静态方法来验证响应的各个方面。

```java
@Test
public void testGetUserById() throws Exception {
    mockMvc.perform(get("/api/users/1"))
        .andExpect(status().isOk()) // 验证 HTTP 状态码是否为 200 OK
        .andExpect(content().contentType(MediaType.APPLICATION_JSON)) // 验证响应类型
        .andExpect(jsonPath("$.id").value(1)) // 使用 JsonPath 验证响应体内容
        .andExpect(jsonPath("$.name").value("John Doe"));
}
```

- `status()`: 验证 HTTP 状态码。
- `content()`: 验证响应体内容或类型。
- `header()`: 验证响应头。
- `cookie()`: 验证 Cookie。
- `jsonPath()`: 针对 JSON 响应，使用类似 XPath 的表达式来提取和验证字段值。

### 4. （可选）打印请求和响应

为了方便调试，你可以使用 `.andDo(print())` 来打印详细的请求和响应信息。

```java
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;

@Test
public void testGetUserById_withLogging() throws Exception {
    mockMvc.perform(get("/api/users/1"))
        .andExpect(status().isOk())
        .andDo(print()); // 打印请求和响应的详细信息
}
```

## 使用场景

`MockMvc` 非常适合用于**控制器层的集成测试**，主要场景包括：

1. **验证请求映射和参数绑定**：确保请求能够正确地路由到目标 Controller 方法，并且请求参数（路径变量、查询参数、请求体等）能够被正确地绑定到方法参数上。
2. **验证输入校验**：测试当传入非法参数时，`@Valid` 等校验注解是否生效，并返回预期的错误响应（如 `400 Bad Request`）。
3. **验证响应状态、头部和内容**：检查控制器是否返回了正确的 HTTP 状态码、`Content-Type` 头部以及符合预期的响应体（JSON, XML, HTML 等）。
4. **测试受保护的端点**：结合 Spring Security Test，可以模拟认证用户和不同权限，测试接口的访问控制逻辑。
5. **无需依赖外部服务**：当控制器依赖于 Service 层时，你可以使用 `@MockBean` 来模拟 Service 的行为，从而将测试范围精确地限制在 Web 层，确保测试的独立性和速度。

总之，`MockMvc` 是测试 Spring MVC Controller 的强大工具，它在不启动完整服务器的情况下，提供了一个接近真实的测试环境，是编写高质量、可靠的 Web 层测试的关键。

## 结合实例：测试中的常见用法解析

让我们通过一个具体的测试类 `FileControllerTest` 来解析在 `MockMvc` 测试中常用的一些辅助工具和方法。

```java
@SpringBootTest // 注意：为了使用 @MockBean，这里需要用 @SpringBootTest
@AutoConfigureMockMvc
class FileControllerTest {

    @Autowired
    MockMvc mockMvc;

    @MockitoBean // Spring Boot 提供的注解，用于在测试上下文中创建一个 Mockito mock 对象
    private AppConfig appConfig;

    private Path tempDir;
    private byte[] fileData;
    private final String fileName = "test.jpg";

    @BeforeEach
    void setUp() throws Exception {
        // 1. 创建临时目录和文件
        tempDir = Files.createTempDirectory("assets");
        fileData = new byte[100];
        // ... (省略文件内容填充)
        Files.write(tempDir.resolve(fileName), fileData); // 2. 使用 resolve

        // 3. 定义 mock 对象的行为
        when(appConfig.getAssetsPath()).thenReturn(tempDir.toString());
    }

    @AfterEach
    void tearDown() throws Exception {
        // 4. 清理资源
        FileSystemUtils.deleteRecursively(tempDir);
    }

    // TODO: implement tests
}
```

### 1. JUnit 生命周期注解: `@BeforeEach` 和 `@AfterEach`

在单元测试中，我们经常需要在每个测试方法运行之前准备好一个“干净”的环境，并在测试结束后进行清理，以避免测试之间的相互干扰。JUnit 5 提供了生命周期注解来帮助我们实现这一点。

- **`@BeforeEach`** (相当于 JUnit 4 的 `@Before`):
  - **作用**: 被此注解标记的方法会在**每一个** `@Test` 方法运行**之前**执行。
  - **用法**: 非常适合用于执行重复性的初始化任务，例如：
    - 创建临时文件或目录。
    - 初始化测试数据。
    - 重置 mock 对象的状态。
  - 在您的代码中，`setUp()` 方法在每个测试开始前都会创建一个新的临时目录和一个文件，确保每个测试都在一个独立、可预测的文件环境中运行。

- **`@AfterEach`** (相当于 JUnit 4 的 `@After`):
  - **作用**: 被此注解标记的方法会在**每一个** `@Test` 方法运行**之后**执行。
  - **用法**: 用于执行清理工作，释放资源，确保测试不会留下任何“垃圾”。例如：
    - 删除临时文件或目录。
    - 关闭数据库连接。
    - 恢复被修改的系统状态。
  - 在您的代码中，`tearDown()` 方法在每个测试结束后都会删除 `setUp()` 中创建的临时目录，保持了测试环境的整洁。

### 2. 路径操作: `Path.resolve()` 方法

`Path.resolve(String other)` 是 Java NIO (`java.nio.file`) 中非常实用的一个方法。

- **作用**: 它用于将一个给定的路径字符串连接到当前的 `Path` 对象上，生成一个新的 `Path`。可以把它想象成在文件系统中执行 `cd` 命令然后查看路径。
- **用法**:
  - 如果 `other` 是一个**相对路径**，`resolve` 会将其附加到当前路径后面。例如，`Path.of("/home/user").resolve("docs")` 会得到 `/home/user/docs`。
  - 如果 `other` 是一个**绝对路径**，`resolve` 会直接返回 `other` 的路径。例如，`Path.of("/home/user").resolve("/etc/config")` 会得到 `/etc/config`。
- 在您的代码中，`tempDir.resolve(fileName)` 将文件名 `test.jpg` 连接到临时目录的路径 `tempDir` 后面，从而得到这个测试文件的完整路径。这是一种健壮且跨平台的构建文件路径的方式。

### 3. Mockito 模拟: `when(...).thenReturn(...)`

在测试控制器时，我们通常希望将测试的焦点**仅限于控制器本身**，而不受其依赖的 Service 或配置类的影响。Mockito 是一个流行的模拟框架，可以创建和配置“假”的对象（mock objects）。`@MockBean` 是 Spring Boot 提供的便利工具，可以轻松地将 Mockito 创建的 mock 对象注入到 Spring 的应用上下文中。

- **作用**: `when(...).thenReturn(...)` 是 Mockito 的核心语法，用于**“打桩” (stubbing)**，即定义一个 mock 对象在被调用时应该如何表现。
- **语法**: `when(mockObject.someMethod(someArgs)).thenReturn(returnValue);`
- **解释**:
  - `when(...)`: 告诉 Mockito 我们准备为一个方法调用进行打桩。括号内是**实际的方法调用**。
  - `thenReturn(...)`: 指定当 `when` 中定义的方法被调用时，应该返回的值。
- 在您的代码中，`when(appConfig.getAssetsPath()).thenReturn(tempDir.toString())` 的意思是：“当测试过程中，有任何代码调用了 `appConfig` 这个 mock 对象的 `getAssetsPath()` 方法时，不要执行真实的方法，而是直接返回 `tempDir.toString()` 的值（即我们刚刚创建的临时目录的路径）”。

这使得您的 `FileController` 在测试时会认为文件存储路径就是您在 `setUp` 方法中动态创建的那个临时目录，从而让您能够精确地控制和验证文件的读写行为，而无需依赖于一个真实的、固定的文件系统路径。

## 深入 MockMvc 与 Mockito 的核心 API

为了编写出高效、精确的测试，我们需要深入理解 `MockMvc` 和 `Mockito` 提供的一些核心方法。

### 1. 结果验证：`andExpect()` 的强大功能

`mockMvc.perform(...)` 返回的 `ResultActions` 对象可以链式调用多个 `.andExpect()`，每个 `.andExpect()` 都用于断言响应的不同方面。

- **`status()`**:
  - **用法**: `andExpect(status().isOk())` 或 `andExpect(status().isNotFound())`
  - **作用**: 专门用于验证 HTTP 响应的状态码。它提供了 `isOk()` (200), `isCreated()` (201), `isBadRequest()` (400), `isNotFound()` (404) 等一系列便捷方法，让断言更具可读性。
  - **场景**: 这是最常用的断言，几乎每个 `MockMvc` 测试都会用它来验证请求是否成功处理。

- **`header()`**:
  - **用法**: `andExpect(header().string("Content-Type", "application/json"))`
  - **作用**: 用于验证响应头（Response Headers）的值。你可以检查某个头是否存在，或者它的值是否符合预期。
  - **场景**: 当你需要确保响应的元数据正确时使用，例如验证 `Content-Type`、`Location` (在创建资源后) 或自定义的 `X-Rate-Limit` 等头部。

- **`jsonPath()`**:
  - **用法**: `andExpect(jsonPath("$.user.name").value("John"))`
  - **作用**: 针对 JSON 格式的响应体，提供了一种强大的“路径表达式”来提取和验证其中的数据。`$` 代表 JSON 的根元素。
  - **场景**: 在测试返回 JSON 数据的 REST API 时必不可少。你可以用它精确地验证响应体中某个字段的值、数组的长度或对象是否存在，而无需将整个 JSON 字符串反序列化为对象再进行断言。

### 2. Mockito 交互：定义行为与验证调用

Mockito 不仅能创建 mock 对象，还能精确地定义其行为并验证它是否被正确地调用。

- **`when(...).thenReturn(...)`**:
  - **用法**: `when(userService.findById(1L)).thenReturn(new User(1L, "test"));`
  - **作用**: 用于“打桩” (stubbing)，即**为有返回值的方法定义一个预设的返回结果**。
  - **场景**: 当你的被测代码（如 Controller）调用了一个依赖组件（如 Service）的方法并需要其返回结果时使用。这可以隔离被测代码，使其不受依赖项内部逻辑的影响。

- **`doNothing().when(...)`**:
  - **用法**: `doNothing().when(emailService).sendWelcomeEmail("test@example.com");`
  - **作用**: 这是专门为 `void`（无返回值）方法设计的打桩方式。它告诉 Mockito：“当这个 `void` 方法被调用时，什么都不要做（不要执行真实逻辑）”。
  - **场景**: 当你需要模拟一个 `void` 方法的调用，但又不希望它的真实逻辑（如发送邮件、写入日志文件）在测试中执行时使用。

- **`verify()`**:
  - **用法**: `verify(userService, times(1)).deleteUser(1L);`
  - **作用**: 用于**验证一个 mock 对象的方法是否被调用过**。它可以精确地检查调用次数、传入的参数等。
  - **场景**: 在测试中，有时我们不仅关心结果，还关心过程。`verify()` 用于确保被测代码确实与它的依赖项发生了预期的交互。例如，验证调用删除用户的接口后，`userService.deleteUser()` 方法是否被**且仅被**调用了一次。

### 3. API 用法总结

| 方法/API | 所属库 | 作用 | 使用场景 |
| :--- | :--- | :--- | :--- |
| `andExpect()` | Spring Test | `MockMvc` 结果断言的容器 | 链式调用，包裹 `status()`, `jsonPath()` 等具体断言。 |
| `status()` | Spring Test | 验证 HTTP 状态码 | 检查响应是 200 OK, 404 Not Found 还是 500 Error。 |
| `header()` | Spring Test | 验证 HTTP 响应头 | 检查 `Content-Type`, `Location` 等响应元数据。 |
| `jsonPath()` | Spring Test / Jayway | 验证 JSON 响应体内容 | 精确断言 REST API 返回的 JSON 数据中的字段值。 |
| `when/thenReturn` | Mockito | 为有返回值的方法打桩 | 模拟依赖项方法的返回结果。 |
| `doNothing` | Mockito | 为 `void` 方法打桩 | 阻止 `void` 方法的真实逻辑在测试中执行。 |
| `verify` | Mockito | 验证方法调用行为 | 检查 mock 对象的方法是否被以预期的方式调用。 |

## 简化测试数据生成：EasyRandom

在编写测试时，创建和填充测试对象（尤其是复杂的 DTO 或实体）是一件繁琐且重复的工作。`EasyRandom` 是一个可以为你随机生成 Java 对象的库，极大地简化了这项工作。

### 引入 EasyRandom

在你的 `pom.xml` (Maven) 或 `build.gradle` (Gradle) 中添加依赖：

**Maven:**

```xml
<dependency>
    <groupId>org.jeasy</groupId>
    <artifactId>easy-random-core</artifactId>
    <version>5.0.0</version>
    <scope>test</scope>
</dependency>
```

### 简单用法

`EasyRandom` 的核心是一个 `EasyRandom` 对象，调用它的 `nextObject()` 方法即可。

```java
import org.jeasy.random.EasyRandom;

public class UserCreationTest {

    private final EasyRandom easyRandom = new EasyRandom();

    @Test
    void testCreateUser() {
        // 随机生成一个 UserDto 对象，所有字段都会被填充上随机值
        UserDto randomUserDto = easyRandom.nextObject(UserDto.class);

        // 你可以根据需要覆盖某些字段的值
        randomUserDto.setEmail("specific-test-email@example.com");

        // 现在，你可以将这个半随机、半定制的 DTO 用于你的 MockMvc 测试
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(randomUserDto)))
            .andExpect(status().isCreated());
    }
}
```

通过 `EasyRandom`，你不再需要手动 `new UserDto()` 然后调用一长串的 `setter` 方法，让测试代码变得更加简洁和专注于业务逻辑的验证。
