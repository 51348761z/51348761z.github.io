---
title: Servlet Notes
date: 2025-10-15 18:46:45 +0800
description: "Notes on HttpServlet and related concepts."
tags: [Java, Servlet]
categories: [java, servlet]
---

## 什么是 Servlet？

Servlet 是一个运行在 Web 服务器或应用服务器（如 Tomcat、Jetty）上的 Java 程序。它充当了客户端（通常是 Web 浏览器）请求与服务器上数据库或应用程序之间的中间层。

简单来说，Servlet 的主要职责是：

1. 接收来自客户端的 HTTP 请求。
2. 处理请求（例如，查询数据库、执行业务逻辑）。
3. 生成动态内容（通常是 HTML，也可以是 JSON、XML 等）作为响应返回给客户端。

Servlet 是 Java EE (现在是 Jakarta EE) 规范的一部分，为开发 Web 应用程序提供了一套标准的 API。

## 在日常项目中如何使用？

在现代的 Java Web 开发中，开发者很少直接编写和管理底层的 Servlet。相反，我们通常会使用建立在 Servlet 技术之上的高级框架，如 Spring MVC 或 Jakarta Faces (JSF)。

这些框架遵循**模型-视图-控制器 (MVC)** 设计模式，并隐藏了许多 Servlet 的底层细节，让开发者可以更专注于业务逻辑。

- **控制器 (Controller)**: 在 Spring MVC 中，使用 `@RestController` 或 `@Controller` 注解的类扮演了控制器的角色。框架的 `DispatcherServlet`（它本身就是一个 Servlet）会接收所有请求，然后根据 URL 将请求分发给相应的控制器方法。

  ```java
  @RestController
  @RequestMapping("/api/users")
  public class UserController {

      @GetMapping("/{id}")
      public User getUserById(@PathVariable Long id) {
          // ... 业务逻辑来获取用户
          return userService.findById(id);
      }
  }
  ```

- **请求和响应**: 框架自动将 `HttpServletRequest` 和 `HttpServletResponse` 对象封装起来。我们可以通过方法参数直接获取请求数据（如 `@PathVariable`, `@RequestParam`, `@RequestBody`），并直接返回 Java 对象（POJO），框架会自动将其序列化为 JSON 或其他格式。

- **配置**: 在过去，Servlet 需要在 `web.xml` 文件中进行配置。而在现代的 Spring Boot 应用中，这通常是自动配置的，我们几乎不需要手动配置 Servlet。

**总结来说**：虽然我们不再频繁地直接编写 `HttpServlet` 的子类，但 Servlet 依然是 Java Web 应用的基石。像 Spring MVC 这样的现代框架，通过在其上构建抽象层，极大地简化了 Web 开发，提高了开发效率。理解 Servlet 的工作原理对于深入排查问题和理解 Web 框架的内部机制非常有帮助。

## 什么是 HttpServletResponse？

`HttpServletResponse` 是 Servlet API 中的一个核心接口，它代表了服务器对客户端 HTTP 请求的响应。当一个 Servlet 处理完请求后，它会使用 `HttpServletResponse` 对象来构建并发送响应回客户端。

这个对象提供了多种方法来设置响应的各个部分，例如：

- **HTTP 状态码**: 使用 `setStatus(int sc)` 来设置响应状态（如 `200 OK`, `404 Not Found`, `500 Internal Server Error`）。
- **HTTP 头部 (Headers)**: 使用 `setHeader(String name, String value)` 或 `addHeader(String name, String value)` 来设置响应头，例如 `Content-Type`, `Cache-Control` 等。
- **响应体 (Body)**: 通过 `getWriter()` 获取一个 `PrintWriter` 来发送文本数据（如 HTML, JSON），或者通过 `getOutputStream()` 获取一个 `ServletOutputStream` 来发送二进制数据（如图片、文件）。

## 什么情况下我们会直接使用 HttpServletResponse 而不是 DTO？

在现代 Spring MVC 等框架中，我们通常从控制器方法返回一个**数据传输对象 (DTO)**，框架会自动将其序列化为 JSON 并设置好 `Content-Type: application/json` 和 `200 OK` 状态码。这在大多数情况下都非常方便。

然而，在某些特定场景下，我们需要更精细地控制 HTTP 响应，这时直接使用 `HttpServletResponse` 就变得必要了。

以下是一些典型情况：

1. **文件下载或数据流**: 当需要让用户下载文件（如 PDF、Excel、ZIP）时，直接返回一个包含所有文件内容的 DTO 会非常消耗内存。正确的做法是，通过 `response.getOutputStream()` 获取输出流，然后将文件内容分块写入流中。同时，还需要设置特定的响应头，如 `Content-Disposition` 来建议浏览器将响应作为附件下载。

    ```java
    @GetMapping("/download-report")
    public void downloadReport(HttpServletResponse response) throws IOException {
        // 设置响应头，告诉浏览器这是一个需要下载的附件
        response.setContentType("application/octet-stream");
        response.setHeader("Content-Disposition", "attachment; filename=\"report.xlsx\"");

        // ... 逻辑来生成或获取文件流
        InputStream fileStream = reportService.generateExcelReport();

        // 将文件流写入响应的输出流
        IOUtils.copy(fileStream, response.getOutputStream());
        response.flushBuffer();
    }
    ```

2. **动态设置 HTTP 状态码和重定向**: 如果你需要根据业务逻辑动态地决定返回哪个 HTTP 状态码，或者执行一个重定向，直接操作 `HttpServletResponse` 会更直接。

    ```java
    @PostMapping("/create-resource")
    public void createResource(@RequestBody Resource resource, HttpServletResponse response) throws IOException {
        Resource newResource = service.create(resource);
        if (newResource != null) {
            // 设置 201 Created 状态码，并通过 Location 头告知新资源的 URL
            response.setStatus(HttpServletResponse.SC_CREATED);
            response.setHeader("Location", "/resources/" + newResource.getId());
        } else {
            // 使用 sendError 发送一个错误状态码和消息
            response.sendError(HttpServletResponse.SC_BAD_REQUEST, "Invalid resource data.");
        }
    }
    ```

3. **操作 Cookies 或自定义头部**: 当你需要对 Cookies 进行复杂的增删改查，或者添加一些非标准的、自定义的 HTTP 响应头时，直接使用 `response.addCookie(...)` 或 `response.setHeader(...)` 是最直接的方式。

**总结来说**：

- **使用 DTO**：当你需要返回结构化的**数据**（通常是 JSON/XML）时，这是最常用、最简洁的方式。框架负责处理序列化和大部分 HTTP 细节。
- **使用 `HttpServletResponse`**：当你需要精细控制 HTTP **协议**本身时，比如处理文件流、重定向、自定义头部、Cookies 或动态状态码时。在这种情况下，你操作的不再仅仅是数据，而是整个 HTTP 响应报文。

## HTTP Range 请求头与断点续传

HTTP `Range` 请求头允许客户端只请求资源的一部分，而不是整个资源。这对于大文件下载（实现**断点续传**）和媒体流（允许用户**拖动视频进度条**）等功能至关重要。

当客户端需要从上次中断的地方继续下载文件时，它可以在 HTTP 请求中加入 `Range` 头，告诉服务器它需要哪一部分数据。

### 工作流程

1. **服务器宣告支持**: 首先，服务器需要在对资源的初始响应（例如，一个 `HEAD` 请求或第一个 `GET` 请求）中包含 `Accept-Ranges: bytes` 头部。这告诉客户端：“我支持按字节范围来请求这个资源。”

2. **客户端请求部分内容**: 客户端发起一个新的请求，并在请求头中包含 `Range` 字段，指定所需的字节范围。`Range` 头的格式有多种：
    - `Range: bytes=0-499`：请求资源的前 500 个字节。
    - `Range: bytes=500-999`：请求第 501 到第 1000 个字节。
    - `Range: bytes=500-`：请求从第 501 个字节开始到资源结束的所有内容。
    - `Range: bytes=-500`：请求资源的最后 500 个字节。

3. **服务器响应部分内容**:
    - 如果服务器能够满足该范围请求，它会返回 `206 Partial Content` 状态码。
    - 响应体中只包含被请求的那部分数据。
    - 响应头中必须包含 `Content-Range` 字段，它说明了这部分数据在整个资源中的位置，例如：`Content-Range: bytes 500-999/12345` (表示发送的是第 500-999 字节，资源总大小为 12345 字节)。
    - 响应头 `Content-Length` 的值是本次传输的字节数（在这个例子中是 500），而不是整个资源的完整大小。

### 在 Servlet/Spring 中处理 Range 请求

处理 `Range` 请求需要手动解析请求头，并精确地控制响应流。这正是需要直接使用 `HttpServletResponse` 的一个典型场景。

以下是一个简化的 Spring MVC 示例，用于处理视频文件的范围请求：

```java
@GetMapping("/stream-video")
public void streamVideo(
        @RequestHeader(value = "Range", required = false) String rangeHeader,
        HttpServletRequest request,
        HttpServletResponse response) throws IOException {

    File videoFile = new File("/path/to/your/video.mp4");
    long fileLength = videoFile.length();

    long rangeStart = 0;
    long rangeEnd = fileLength - 1;

    // 如果请求头中包含 Range
    if (rangeHeader != null && rangeHeader.startsWith("bytes=")) {
        String[] ranges = rangeHeader.substring("bytes=".length()).split("-");
        rangeStart = Long.parseLong(ranges[0]);
        // 如果 Range 包含了结束位置
        if (ranges.length > 1) {
            rangeEnd = Long.parseLong(ranges[1]);
        }
    }

    long contentLength = rangeEnd - rangeStart + 1;

    // 设置响应头
    response.setContentType("video/mp4");
    response.setHeader("Accept-Ranges", "bytes");
    response.setDateHeader("Last-Modified", videoFile.lastModified());

    // 如果是范围请求
    if (rangeHeader != null) {
        response.setStatus(HttpServletResponse.SC_PARTIAL_CONTENT);
        response.setHeader("Content-Range", "bytes " + rangeStart + "-" + rangeEnd + "/" + fileLength);
    } else { // 如果是完整请求
        response.setStatus(HttpServletResponse.SC_OK);
    }
    response.setHeader("Content-Length", String.valueOf(contentLength));


    // 使用 RandomAccessFile 来读取文件的指定部分
    try (RandomAccessFile randomAccessFile = new RandomAccessFile(videoFile, "r")) {
        randomAccessFile.seek(rangeStart);
        byte[] buffer = new byte[4096];
        int bytesRead;
        long bytesToWrite = contentLength;

        OutputStream outputStream = response.getOutputStream();
        while ((bytesRead = randomAccessFile.read(buffer, 0, (int) Math.min(buffer.length, bytesToWrite))) != -1 && bytesToWrite > 0) {
            outputStream.write(buffer, 0, bytesRead);
            bytesToWrite -= bytesRead;
        }
        outputStream.flush();
    }
}
```

这个例子展示了如何通过解析 `Range` 头和设置 `Content-Range`、`206` 状态码等，来实现一个支持拖动播放的视频流服务器端点。

## 什么是 `RandomAccessFile`？

`java.io.RandomAccessFile` 是 Java I/O API 中的一个特殊类，它允许对文件进行非顺序的、即“随机”的读写操作。

与 `FileInputStream` 和 `FileOutputStream` 这种只能从头到尾顺序读写的流不同，`RandomAccessFile` 的核心特点是它内部维护一个文件指针（file pointer）。你可以自由地移动这个指针到文件中的任意位置，然后从该位置开始进行读取或写入。

### 主要特点

1. **自由定位**：通过 `seek(long pos)` 方法，可以立即将文件指针移动到指定的字节位置。
2. **读写兼备**：可以在同一个实例上进行读取（`read()`）和写入（`write()`）操作，这取决于打开文件时指定的模式（如 `"r"` 为只读，`"rw"` 为读写）。
3. **直接操作文件**：它不属于 `InputStream` 或 `OutputStream` 的子类，直接继承自 `Object`。

### 使用场景

`RandomAccessFile` 的独特能力使其在以下场景中非常有用：

1. **处理 HTTP Range 请求**：这是最典型的应用场景之一。当需要实现断点续传或视频拖动播放时，服务器可以根据客户端请求的 `Range` 头，使用 `seek()` 方法快速定位到文件的指定部分，然后只读取并发送那一段数据，极大地提高了大文件传输的效率和灵活性。
2. **文件部分修改**：当你需要修改一个大文件中的一小部分内容时，无需将整个文件读入内存再写回。可以直接定位到要修改的位置，然后覆盖写入新数据。
3. **访问固定记录大小的文件**：如果一个文件由大小相同的记录组成，你可以轻松地计算出任何一条记录的起始位置（`record_index * record_size`），然后使用 `seek()` 直接跳转到该记录进行读写。

### 基本使用方法和示例

`RandomAccessFile` 的使用通常遵循“打开 -> 定位 -> 读/写 -> 关闭”的模式。

- **构造函数**: `new RandomAccessFile(File file, String mode)` 或 `new RandomAccessFile(String path, String mode)`。
  - `mode`: `"r"` (只读), `"rw"` (读写), `"rws"` (读写，并同步文件内容和元数据到物理存储), `"rwd"` (读写，并同步文件内容到物理存储)。
- **核心方法**:
  - `seek(long pos)`: 移动文件指针到指定位置。
  - `read(byte[] b)`: 从当前指针位置读取数据到字节数组。
  - `write(byte[] b)`: 在当前指针位置写入字节数组。
  - `length()`: 获取文件的总长度（以字节为单位）。
  - `getFilePointer()`: 获取当前文件指针的位置。
  - `close()`: 关闭文件并释放资源（强烈建议使用 `try-with-resources` 语句来自动管理）。

#### 简单示例

以下代码演示了如何使用 `RandomAccessFile` 读取文件特定部分的内容。

```java
import java.io.File;
import java.io.IOException;
import java.io.RandomAccessFile;
import java.nio.charset.StandardCharsets;

public class RandomAccessFileExample {
    public static void main(String[] args) {
        // 假设有一个 example.txt 文件，内容为 "Hello, World! This is a test."
        File file = new File("example.txt");

        // 使用 try-with-resources 确保文件被正确关闭
        try (RandomAccessFile raf = new RandomAccessFile(file, "r")) {
            // 1. 定位到第 7 个字节（'W'的位置）
            raf.seek(7);

            // 2. 读取 5 个字节 ("World")
            byte[] buffer = new byte[5];
            int bytesRead = raf.read(buffer);

            if (bytesRead != -1) {
                System.out.println("Read " + bytesRead + " bytes: " + new String(buffer, StandardCharsets.UTF_8));
            }

            // 3. 获取当前指针位置
            System.out.println("Current pointer position: " + raf.getFilePointer()); // 输出 12

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 常见的 `HttpServletResponse.setHeader` 取值与应用场景

在处理 HTTP 响应时，除了设置状态码和响应体之外，设置适当的响应头（response headers）对客户端行为、缓存策略、安全和文件下载等都有重要影响。下面列出了一些常见的响应头、它们的含义、典型使用场景以及示例。

### 1. Content-Type

- 含义：声明响应体的媒体类型（MIME type），例如 `text/html`, `application/json`, `image/png`, `video/mp4` 等。

- 场景：几乎所有响应都应该设置。框架在返回 DTO 时通常会自动设置为 `application/json`。

- 示例：

```java
response.setHeader("Content-Type", "application/json; charset=UTF-8");
```

### 2. Content-Length

- 含义：表示响应体的字节长度（当可预知时）。

- 场景：对于静态文件或分块传输前已知长度的部分（如 Range 请求），设置 `Content-Length` 可帮助客户端知道接收长度。注意使用 `Transfer-Encoding: chunked` 时通常不设置此头。

- 示例：

```java
response.setHeader("Content-Length", String.valueOf(contentLength));
```

### 3. Content-Range

- 含义：用于部分内容响应（`206 Partial Content`），描述发送的字节范围和资源总长度。例如：`bytes 500-999/12345`。

- 场景：实现断点续传或流媒体（支持拖动播放）时使用。

- 示例：

```java
response.setStatus(HttpServletResponse.SC_PARTIAL_CONTENT);
response.setHeader("Content-Range", "bytes " + rangeStart + "-" + rangeEnd + "/" + fileLength);
```

### 4. Cache-Control

- 含义：控制缓存策略，如 `no-cache`, `no-store`, `max-age=3600` 等。

- 场景：静态资源（图片、脚本、样式）通常设置较长的缓存时间；敏感数据或经常变化的资源应禁用缓存或短期缓存。

- 示例：

```java
response.setHeader("Cache-Control", "max-age=3600, public");
// 或禁止缓存
response.setHeader("Cache-Control", "no-store");
```

### 5. Content-Disposition

- 含义：指导浏览器如何处理响应体：是内联显示(`inline`)还是作为附件下载(`attachment`)。通常也用于建议下载文件名。

- 场景：文件下载时必须设置以提示浏览器弹出下载对话框并指定文件名。

- 示例：

```java
response.setHeader("Content-Disposition", "attachment; filename=\"report.xlsx\"");
```

> 注意：为处理中文或特殊字符，最好对文件名做 URL 编码或使用 `filename*`（RFC 5987）。

### 6. Accept-Ranges

- 含义：告知客户端服务器是否支持范围请求（通常为 `bytes`）。

- 场景：实现断点续传和媒体流时返回 `Accept-Ranges: bytes` 可以让客户端知道可发起 `Range` 请求。

- 示例：

```java
response.setHeader("Accept-Ranges", "bytes");
```

### 7. Last-Modified

- 含义：资源的最后修改时间（HTTP date 格式，服务器可使用 `setDateHeader`）。

- 场景：配合 `If-Modified-Since` 使用，便于实现条件请求（返回 304 Not Modified）以节省带宽。

- 示例：

```java
response.setDateHeader("Last-Modified", file.lastModified());
```

### 8. ETag

- 含义：资源的实体标签（一个唯一标识符，通常是内容哈希或基于修改时间的标识）。

- 场景：同样用于条件请求，客户端可以发送 `If-None-Match`。如果 ETag 未改变，服务器返回 `304 Not Modified`。

- 示例：

```java
String etag = "W/\"" + file.length() + "-" + file.lastModified() + "\""; // 简化示例
response.setHeader("ETag", etag);
```

### 9. Location

- 含义：用于重定向，指示客户端新的资源 URL。通常与 3xx 状态码一起使用（如 `302 Found`, `301 Moved Permanently`, `201 Created`）。

- 场景：创建资源后返回 `201 Created` 并在 `Location` 头中给出新资源地址；或执行临时/永久重定向时使用。

- 示例：

```java
response.setStatus(HttpServletResponse.SC_CREATED);
response.setHeader("Location", "/resources/" + newResource.getId());
```

### 10. Set-Cookie

- 含义：设置或更新客户端 Cookie。推荐使用 `javax.servlet.http.Cookie` 对象和 `addCookie` 方法来处理复杂的 cookie 属性（如 `HttpOnly`, `Secure`, `SameSite`）。

- 场景：会话管理、记住登录、跟踪偏好等。

- 示例：

```java
Cookie cookie = new Cookie("sessionId", sessionId);
cookie.setHttpOnly(true);
cookie.setSecure(true);
cookie.setPath("/");
response.addCookie(cookie);
```

## `HttpServletResponse.setStatus` 与常用 HTTP 状态码

`HttpServletResponse.setStatus(int sc)` 用于设置响应的 HTTP 状态码。状态码告诉客户端服务器对请求的处理结果（成功、重定向、客户端错误或服务器错误等）。在复杂的响应场景下，你可能还会使用 `sendError(int)`、`sendError(int, String)` 或 `sendRedirect(String)` 等方法来配合使用。

### 常用方法对比

- `response.setStatus(int sc)`：仅设置状态码，通常与自定义响应体或头部一起使用。例如设置 `201 Created` 并通过 `Location` 告知新资源地址。

- `response.sendError(int sc)`：发送错误状态码并包含一个默认错误页面（容器会短路并提交响应）。可以带上消息：`sendError(int, String)`。

- `response.sendRedirect(String location)`：设置 302（或在 Servlet 规范中可选为 307/303，容器通常会处理）并在 `Location` 头里写入目标 URL，然后立即提交响应。

> 在 Spring MVC 中，通常更推荐抛出 `ResponseStatusException`、使用 `ResponseEntity` 或框架提供的重定向/异常处理方式，但在需要直接控制原始响应报文时仍会使用上述原生方法。

### 常见状态码（选取常用并带示例）

- 200 OK — 请求成功且有返回内容（默认）。
  - 示例：`response.setStatus(HttpServletResponse.SC_OK);`

- 201 Created — 资源已创建。通常返回 `Location` 头指向新资源。
  - 示例：

```java
response.setStatus(HttpServletResponse.SC_CREATED);
response.setHeader("Location", "/resources/" + newResource.getId());
```

- 204 No Content — 请求成功，但无响应体（常用于 DELETE 等操作成功后）。
  - 示例：`response.setStatus(HttpServletResponse.SC_NO_CONTENT);`

- 301 Moved Permanently / 302 Found / 303 See Other — 重定向相关（长期/临时/方法切换）。
  - 使用：`response.sendRedirect("/new-url")`（常用，容器会设置 Location 并返回 302）。

- 304 Not Modified — 客户端缓存有效，服务器无需返回内容（与 `If-Modified-Since` / `If-None-Match` 配合使用）。
  - 使用场景：当 `ETag` 或 `Last-Modified` 未变更时返回 `304`，让客户端使用本地缓存。
  - 示例：

```java
if (clientETag.equals(serverEtag)) {
    response.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
    return; // 不返回内容
}
```

- 400 Bad Request — 请求参数或格式错误。
  - 示例：`response.sendError(HttpServletResponse.SC_BAD_REQUEST, "Missing required field 'name'");`

- 401 Unauthorized — 需要认证或认证失败（通常由认证中间件/过滤器处理）。

- 403 Forbidden — 已认证但没有权限访问。

- 404 Not Found — 资源不存在。
  - 示例：`response.sendError(HttpServletResponse.SC_NOT_FOUND, "Not found");`

- 405 Method Not Allowed — HTTP 方法不允许。

- 409 Conflict — 资源冲突（例如并发更新冲突或唯一约束违反）。

- 429 Too Many Requests — 请求速率限制触发。

- 500 Internal Server Error — 服务器异常或未处理的错误。
  - 示例：`response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR, "Unexpected error");`

- 502/503/504 — 反向代理或上游服务相关错误（bad gateway / service unavailable / gateway timeout）。

### 实践建议与注意事项

1. 优先使用框架的抽象（如 Spring 的 `ResponseEntity`、异常处理器）来统一管理状态码和响应体；直接操作 `HttpServletResponse` 更适合文件流、重定向或需要微调 HTTP 报文的场景。

2. 在返回 `201 Created` 时，最好同时设置 `Location` 头以告知新资源地址，且返回响应体（可选）包含新资源信息。

3. `sendError` 会委托给容器生成错误页面并提交响应（不会像 `setStatus` 一样允许你继续写入自定义 body），因此在需要自定义错误 JSON 时，可以手动设置 `setStatus` 并写入自定义错误体。

4. 对于 API，尽量保持语义化的状态码（例如：验证错误用 400、权限相关用 401/403、资源缺失用 404、业务冲突用 409），这有助于客户端正确处理不同情况。

---
