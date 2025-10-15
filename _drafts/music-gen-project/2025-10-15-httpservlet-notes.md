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
