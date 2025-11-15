---
title: Redis Stream Introduction and Quickstart
date: 2025-11-01T05:24:05+08:00
categories: [Redis, Stream]
tags: [Redis, Stream]
---

## 什么是 Redis Stream？

Redis Stream 是一个仅追加的日志数据结构。它就像一个日志文件，但功能更强大。你可以将新条目（消息）添加到末尾，每个条目都会获得一个唯一的、基于时间的 ID。

主要特性：

- **仅追加（Append-only）**: 你只能在末尾添加新消息。
- **持久化（Persistent）**: 数据存储在内存中，但可以被持久化。
- **消费组（Consumer Groups）**: 允许多个客户端协同处理来自同一个流的消息，确保每条消息只被一个客户端处理。这实现了负载均衡和容错。
- **时间序列数据（Time-series Data）**: 自动生成的、基于时间的 ID 使其非常适合存储时间序列数据。

## 使用 Redis Stream 实现生产者-消费者模式

该模式涉及两种类型的参与者：

1. **生产者（Producers）**: 向流中写入事件/消息的应用程序。
2. **消费者（Consumers）**: 从流中读取并处理消息的应用程序。

`GenerationTaskHandlerService` 类就是一个完美的 **消费者** 示例。

### 实践原理：代码解析

让我们分解 `GenerationTaskHandlerService`，看看这些概念是如何应用的。

#### 1. 初始化 (`@PostConstruct`)

当应用程序启动时，`startTaskConsumer` 方法会被调用。

- **创建消费组**: 它首先确保流的消费组存在。

    ```java
    redisTemplate.opsForStream().createGroup(STREAM_KEY, ReadOffset.from("0"), GROUP_NAME);
    ```

  - `STREAM_KEY`: 要监听的流的名称 (例如, `generation:GenerationTaskHandlerService`)。
  - `GROUP_NAME`: 工作者团队的名称 (例如, `generation-group`)。
  - `ReadOffset.from("0")`: 如果消费组是新创建的，它应该能够从流的最初始位置读取所有消息。
- **启动后台线程**: 它将 `taskProcessingLoop` 任务提交给一个 `ExecutorService`。这一点至关重要，因为监听过程是一个阻塞式的无限循环；在后台运行它可以防止主应用程序被冻结。

#### 2. 处理循环 (`taskProcessingLoop`)

这是消费者的核心。它是一个无限的 `while(running)` 循环，持续监听消息。

- **阻塞式读取**: 最重要的一行是 `read` 命令。

    ```java
    List<ObjectRecord<String, GenerationTaskMessage>> records = redisTemplate.opsForStream().read(...)
    ```

    它通过 `StreamReadOptions` 进行配置以提高效率：
  - `block(Duration.ofMillis(5000))`: 这告诉 Redis：“如果没有消息，最多等待 5 秒再响应。” 这远比不断地询问“有新消息了吗？”要高效得多。
  - `count(1)`: 一次处理一条消息。
- **消费者与偏移量**: 读取命令需要知道是 *谁* 在读取以及从 *哪里* 开始读。
  - `Consumer.from(GROUP_NAME, CONSUMER_NAME)`: 标识该消费组内的这个特定工作者。
  - `StreamOffset.create(STREAM_KEY, ReadOffset.lastConsumed())`: 这个特殊的偏移量 (`>`) 告诉 Redis：“把我的消费组里还没有人看过的下一条消息给我。”

#### 3. 消息处理与确认

- **处理**: 当收到一条消息时，代码获取其内容 (`record.getValue()`)。
- **确认 (最关键的一步)**:

    ```java
    redisTemplate.opsForStream().acknowledge(STREAM_KEY, GROUP_NAME, recordId);
    ```

    这告诉 Redis：“我已经成功处理完这条消息了。”
  - **为什么这很重要？** 当一个消费者读取一条消息时，Redis 会为该消费者将该消息放入一个“待处理条目列表”（Pending Entries List, PEL）中。如果消费者在确认之前崩溃，消息会保留在 PEL 中。超时后，组内的另一个消费者可以认领并重新处理这条待处理的消息，从而确保 **没有消息丢失**。这提供了“至少一次”的投递保证。

#### 4. 关闭 (`@PreDestroy`)

当应用程序关闭时，`stopTaskConsumer` 方法会被调用。

- `this.running = false;`: 这会优雅地停止 `while` 循环。
- `executor.shutdownNow();`: 这会强制中断后台线程，以确保干净、立即地关闭。
