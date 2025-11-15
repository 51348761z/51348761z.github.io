---
title: Minio Quickstart Guide
date: 2025-10-31T18:03:20+08:00
categories: [Java, Minio]
tags: [Java, Minio, Object Storage]
---

## 1. 什么是 Minio？

Minio 是一个开源的、高性能的对象存储服务器。它基于 Apache License v2.0 发布，API 与 Amazon S3 (Simple Storage Service) 兼容。这意味着你可以将 Minio 作为 S3 的私有化替代方案，轻松地在自己的服务器或私有云环境中部署。

**核心理念**：简单、高性能、可扩展。你可以用它来存储海量的非结构化数据，如图片、视频、日志文件、备份和容器镜像等。

## 2. 核心特性与使用场景

### 核心特性

- **S3 兼容**：几乎所有支持 S3 的工具和库都可以无缝与 Minio 对接。
- **高性能**：针对大文件和高并发场景进行了优化，读写性能优异。
- **可扩展性**：支持横向扩展，可以通过添加更多服务器来构建一个庞大的分布式存储集群。
- **云原生**：为容器化和 Kubernetes 等现代云原生环境设计，易于部署和管理。
- **数据保护**：通过纠删码（Erasure Code）技术，即使部分硬盘损坏，也能保证数据安全。

### 典型使用场景

1. **私有云存储**：为企业内部应用提供可靠、可控的对象存储服务，替代昂贵的公有云存储。
2. **图片/视频服务**：作为网站或 App 的后端存储，用于存放用户上传的图片、视频等媒体文件。
3. **数据备份与归档**：存储数据库备份、日志文件或其他需要长期归档的数据。
4. **大数据与 AI**：作为数据湖的存储层，为 Spark、Hadoop、TensorFlow 等大数据和机器学习框架提供数据源。
5. **CI/CD 制品库**：存储持续集成/持续部署过程中产生的构建产物（Artifacts）。

## 3. 快速上手 (Java)

### 3.1. 启动 Minio 服务 (使用 Docker)

在本地启动一个 Minio 服务最简单的方式是使用 Docker。创建一个 `docker-compose.yml` 文件：

```yaml
version: '3.7'
services:
  minio:
    image: minio/minio:latest
    container_name: minio
    ports:
      - "9000:9000"  # API port
      - "9001:9001"  # Console UI port
    environment:
      - MINIO_ROOT_USER=minioadmin      # 管理员账号
      - MINIO_ROOT_PASSWORD=minioadmin  # 管理员密码
    command: server /data --console-address ":9001"
    volumes:
      - ./minio_data:/data
```

然后在该目录下运行 `docker-compose up -d` 即可启动。你可以通过浏览器访问 `http://localhost:9001` 查看 Minio 的管理控制台。

### 3.2. 引入 Java SDK

在你的 Maven 项目中，添加 Minio Java SDK 依赖：

```xml
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>8.5.10</version> <!-- 建议使用最新稳定版 -->
</dependency>
```

### 3.3. 初始化 MinioClient

创建一个 `MinioClient` 实例来与 Minio 服务进行交互。

```java
import io.minio.MinioClient;

public class MinioConfig {
    public MinioClient minioClient() {
        return MinioClient.builder()
                .endpoint("http://localhost:9000") // Minio 服务地址
                .credentials("minioadmin", "minioadmin") // 管理员账号密码
                .build();
    }
}
```

## 4. 常用操作示例代码

以下是一些基于 Minio Java SDK 的常用操作示例。

```java
// 假设 minioClient 已经通过上面的配置初始化好了
MinioClient minioClient = ...; 
String bucketName = "my-bucket";
String objectName = "my-object.txt";
String filePath = "/path/to/your/file.txt";

// --- Bucket (存储桶) 操作 ---

// 1. 检查存储桶是否存在
boolean found = minioClient.bucketExists(BucketExistsArgs.builder().bucket(bucketName).build());
if (!found) {
    // 2. 创建一个新的存储桶
    minioClient.makeBucket(MakeBucketArgs.builder().bucket(bucketName).build());
    System.out.println("Bucket '" + bucketName + "' created.");
} else {
    System.out.println("Bucket '" + bucketName + "' already exists.");
}

// --- Object (对象) 操作 ---

// 3. 上传文件
minioClient.uploadObject(
    UploadObjectArgs.builder()
        .bucket(bucketName)
        .object(objectName) // 在存储桶中的对象名称
        .filename(filePath) // 本地文件路径
        .build());
System.out.println("File uploaded successfully.");

// 4. 下载文件
minioClient.downloadObject(
    DownloadObjectArgs.builder()
        .bucket(bucketName)
        .object(objectName)
        .filename("downloaded-" + objectName) // 下载后保存的文件名
        .build());
System.out.println("File downloaded successfully.");

// 5. 生成一个带签名的临时访问 URL (有效期 7 天)
String url = minioClient.getPresignedObjectUrl(
    GetPresignedObjectUrlArgs.builder()
        .method(Method.GET)
        .bucket(bucketName)
        .object(objectName)
        .expiry(7, TimeUnit.DAYS)
        .build());
System.out.println("Presigned URL: " + url);

// 6. 删除对象
minioClient.removeObject(
    RemoveObjectArgs.builder()
        .bucket(bucketName)
        .object(objectName)
        .build());
System.out.println("Object deleted successfully.");
```

## 5. MinIO 控制台（mc）常用命令

MinIO 官方提供了一个命令行客户端 `mc`（MinIO Client），用于在终端中执行与 MinIO / S3 兼容服务的常见管理与数据操作。下面按场景整理常用命令与示例：

### 5.1. 配置 alias（主机别名）

在使用之前，先添加一个主机别名（alias），便于后续命令引用：

```bash
mc alias set myminio http://localhost:9000 minioadmin minioadmin
# 查看已配置的别名
mc alias list
```

### 5.2. 列表与查看

```bash
# 列出所有主机/存储桶
mc ls
# 列出指定别名下的存储桶
mc ls myminio
# 列出存储桶中的对象（支持 --recursive）
mc ls --recursive myminio/my-bucket
# 查看对象元数据
mc stat myminio/my-bucket/object.txt
# 打印对象内容（小文件）
mc cat myminio/my-bucket/object.txt
```

### 5.3. 存储桶操作（创建 / 删除）

```bash
# 创建存储桶
mc mb myminio/my-bucket
# 强制删除存储桶（会一并删除其中对象）
mc rb --force myminio/my-bucket
```

### 5.4. 上传、下载、移动与复制

```bash
# 上传文件到存储桶
mc cp /local/path/file.txt myminio/my-bucket/file.txt
# 从存储桶下载文件到本地
mc cp myminio/my-bucket/file.txt /local/path/
# 移动/重命名对象
mc mv myminio/my-bucket/oldname.txt myminio/my-bucket/newname.txt
```

### 5.5. 同步 / 镜像（常用于批量上传或备份）

```bash
# 本地目录镜像到存储桶（增量、保持目录结构）
mc mirror /local/dir myminio/my-bucket
# 支持 --overwrite, --remove, --watch 等选项
mc mirror --overwrite /local/dir myminio/my-bucket
```

### 5.6. 查找与批量操作

```bash
# 在存储桶中查找满足条件的对象（示例：查找后缀为 .log 的文件）
mc find myminio/my-bucket --name "*.log" --older-than 30d
# 对查找到的对象做批量删除或复制（配合脚本或 mc 的输出）
```

### 5.7. 生成临时共享链接

```bash
# 生成一个临时下载链接（示例：7 天有效期）
mc share download --expire 7d myminio/my-bucket/object.txt
```

### 5.8. 存储桶访问策略（Bucket policy）

```bash
# 设置存储桶为允许匿名下载（readonly / download）
mc policy set download myminio/my-bucket
# 查看存储桶策略
mc policy info myminio/my-bucket
# 取消策略（恢复私有）
mc policy set none myminio/my-bucket
```

### 5.9. 管理员命令（需要对 alias 指向的服务有管理员权限）

```bash
# 查看 MinIO 服务信息
mc admin info myminio
# 添加/删除用户（示例）
mc admin user add myminio alice secretpassword
mc admin user remove myminio alice
# 为用户绑定策略（或创建策略后绑定）
mc admin policy attach myminio readwrite --user alice
# 查看服务运行状态或重新启动
mc admin service restart myminio
# 查看或下载 server 日志/trace（排查问题时使用）
mc admin trace myminio --start
```

### 5.10. 常用选项速览

- --recursive：递归操作目录或存储桶
- --insecure：在 HTTPS 验证证书时忽略（仅测试环境慎用）
- --quiet：减少输出，批量脚本常用
- --json：以 JSON 输出，便于脚本解析

小提示：大部分 `mc` 命令支持补全和 `--help`，例如 `mc cp --help` 或 `mc admin --help`，用于查看该子命令的更多选项与用法示例。
