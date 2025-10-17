---
title: 'Music-Gen Project: MySQL configuration'
date: 2025-10-09 17:44:54 +0800
categories: [Java, Project]
tags: [springboot, mysql, java]
mermaid: true
description: "This article discusses the configuration of MySQL for the Music-Gen project."
---


This article discusses the configuration of MySQL for the Music-Gen project.

### 1. User-related Table

```sql
CREATE DATABASE music_gen_db;
USE music_gen_db;
CREATE TABLE `user_info` (
  `user_id` varchar(32) NOT NULL COMMENT 'User ID',
  `email` varchar(50) DEFAULT NULL COMMENT 'Email',
  `nick_name` varchar(20) DEFAULT NULL COMMENT 'Nickname',
  `avatar` varchar(50) DEFAULT NULL COMMENT 'User avatar',
  `password` varchar(32) DEFAULT NULL COMMENT 'Password',
  `status` tinyint(1) DEFAULT NULL COMMENT 'Status',
  `create_time` datetime DEFAULT NULL COMMENT 'Creation time',
  `last_login_time` datetime DEFAULT NULL COMMENT 'Last login time',
  `credits` int(11) DEFAULT '0' COMMENT 'Points for user activities',
  PRIMARY KEY (`user_id`) USING BTREE,
  UNIQUE KEY `idx_key_email` (`email`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='User Information';
```

Some SQL clause explanations are provided below.

#### Why use `USING BTREE`?

You might notice the `USING BTREE` clause for the primary and unique keys. Here’s a brief explanation:

- **What it is**: `BTREE` stands for B-Tree, which is the default and most common index data structure in MySQL's InnoDB storage engine.
- **Why it's used**: B-Tree indexes are highly efficient for a wide variety of queries. They store data in a sorted order, which makes them excellent for:
  - **Exact Lookups**: Quickly finding a row by its exact value (e.g., `WHERE user_id = 'some-id'`). This is essential for primary keys.
  - **Range Queries**: Efficiently retrieving rows within a range (e.g., `WHERE create_time > '2025-01-01'`).
- **Is it necessary?**: No. Since `BTREE` is the default for InnoDB, explicitly stating `USING BTREE` is optional. The database would use it anyway. However, it can be useful for code clarity.

#### What is `ROW_FORMAT=DYNAMIC`?

This setting defines how MySQL stores row data on disk. `DYNAMIC` is the modern default for the InnoDB engine and is generally the best choice.

- **What it does**: It optimizes storage for variable-length columns (like `VARCHAR` or `TEXT`).
- **Why it's used**: Instead of trying to cram the beginning of large text or blob data into the index page itself (like the older `COMPACT` format did), `DYNAMIC` stores the entire large column in separate "overflow pages." This keeps the primary index pages lean and fast, improving overall query performance because more index entries can fit into memory.
- **Is it necessary?**: In modern MySQL (5.7+), `DYNAMIC` is the default. Specifying it is good practice for ensuring consistent behavior across different server configurations.

## 2. Music-related Table

### Music Information Table

```sql
CREATE TABLE `tb_music_info` (
    `music_id` varchar(32) NOT NULL COMMENT 'Unique identifier for the music track',
    `user_id` varchar(32) NOT NULL COMMENT 'ID of the user who created the music',
    `task_id` varchar(32) NOT NULL COMMENT 'ID of the generation task from the AI service',
    `creation_id` varchar(32) NOT NULL COMMENT 'Internal creation identifier',
    `music_title` varchar(30) DEFAULT NULL COMMENT 'Title of the music track',
    `cover` varchar(150) DEFAULT NULL COMMENT 'URL to the cover image',
    `audio_path` varchar(150) DEFAULT NULL COMMENT 'Path or URL to the audio file',
    `duration` int(11) DEFAULT NULL COMMENT 'Duration of the music in seconds',
    `lyrics` text COMMENT 'Lyrics of the music',
    `play_count` int(11) DEFAULT '0' COMMENT 'Number of times the music has been played',
    `like_count` int(11) DEFAULT '0' COMMENT 'Number of likes the music has received',
    `recommended` tinyint(1) DEFAULT '0' COMMENT 'Recommendation status (0: Not recommended, 1: Recommended)',
    `create_time` datetime DEFAULT NULL COMMENT 'Timestamp when the music record was created',
    `music_status` tinyint(1) DEFAULT '0' COMMENT 'Generation status (0: Generating, 1: Generation complete)',
    `music_type` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'Type of music (0: With vocals, 1: Instrumental)',
    `version` bigint(20) DEFAULT '0' COMMENT 'Version number for optimistic locking',
    PRIMARY KEY (`music_id`) USING BTREE,
    UNIQUE KEY `uk_task_id` (`task_id`) USING BTREE COMMENT 'Ensure task ID from the AI service is unique'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='Stores information about generated music tracks';
```

### Music Generation Information Table

```sql
CREATE TABLE `tb_music_generation_info` (
  `creation_id` varchar(32) NOT NULL COMMENT 'Unique identifier for the creation task',
  `user_id` varchar(32) NOT NULL COMMENT 'ID of the user who initiated the task',
  `prompt` varchar(500) NOT NULL COMMENT 'Prompt used for music generation',
  `lyrics` varchar(1500) DEFAULT NULL COMMENT 'Lyrics provided for the music',
  `model` varchar(32) NOT NULL COMMENT 'AI model used for generation',
  `music_type` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'Type of music (0: With vocals, 1: Instrumental)',
  `generation_mode` tinyint(1) DEFAULT NULL COMMENT 'Generation mode (0: Simple, 1: Expert)',
  `settings` varchar(500) DEFAULT NULL COMMENT 'Additional settings for the generation task (e.g., JSON format)',
  `create_time` datetime DEFAULT NULL COMMENT 'Timestamp when the task was created',
  PRIMARY KEY (`creation_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='Stores information about music generation tasks';
```

### Music Operation Table

```sql
CREATE TABLE `tb_music_operation_info` (
  `action_id` int(32) NOT NULL AUTO_INCREMENT COMMENT 'Unique identifier for the action',
  `music_id` varchar(32) NOT NULL COMMENT 'ID of the music being acted upon',
  `music_owner_user_id` varchar(32) DEFAULT NULL COMMENT 'ID of the user who owns the music',
  `user_id` varchar(32) NOT NULL COMMENT 'ID of the user performing the action',
  `action_type` tinyint(1) DEFAULT NULL COMMENT 'Type of action (e.g., 1: Like)',
  PRIMARY KEY (`action_id`) USING BTREE,
  UNIQUE KEY `idx_key_user_music_id` (`music_id`,`user_id`) USING BTREE COMMENT 'Ensures a user can perform a specific action on a piece of music only once',
  KEY `idx_user_id` (`user_id`) USING BTREE COMMENT 'Index for quick lookups of actions by user'
) ENGINE=InnoDB AUTO_INCREMENT=105 DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='Stores user operations on music, such as likes';
```

## 4. Java Data Type Mappings

When mapping SQL data types to Java data types, consider the following recommendations for optimal compatibility and performance:

| **SQL Data Type** | **Recommended Java Data Type** | **Reason**                                                                                        | **** |
|-------------------|--------------------------------|---------------------------------------------------------------------------------------------------|------|
| **INT**           | `Integer`                      | Precise 32-bit integer mapping, and can handle `NULL` values.                                     |      |
| **TINYINT(1)**    | `Integer` or `Boolean`         | `Integer` is more versatile; `Boolean` is semantically clearer if it's explicitly a boolean flag. |      |
| **BIGINT**        | `Long`                         | Precise 64-bit integer mapping, and can handle `NULL` values.                                     |      |
| **BINARY(16)**    | `java.util.UUID`               | Type-safe, semantically clear, and well-supported by frameworks.                                  |      |
| **VARCHAR**       | `String`                       | Standard string mapping.                                                                          |      |
| **DATETIME**      | `java.time.LocalDateTime`      | The standard way to handle date and time in modern Java.                                          |      |

## 5. Product-related tables

### Product Information Table

```sql
CREATE TABLE `tb_product_info`
(
    `product_id`   BIGINT(20)     NOT NULL AUTO_INCREMENT COMMENT 'Product ID',
    `product_name` VARCHAR(100)   NOT NULL COMMENT 'Product Name/Title',
    `description`  VARCHAR(255) DEFAULT NULL COMMENT 'Product Description',
    `price`        DECIMAL(10, 2) NOT NULL COMMENT 'Product Price',
    `cover`        VARCHAR(150) DEFAULT NULL COMMENT 'Product Cover Image URL',
    `create_at`    DATETIME     DEFAULT NULL COMMENT 'Creation Timestamp',
    `category`     VARCHAR(50)  DEFAULT NULL COMMENT 'Product Category',
    `status`       VARCHAR(20)  DEFAULT 'DRAFT' COMMENT 'Product Status',
    `credits`      INT(11)      DEFAULT 0 COMMENT 'Credits required to redeem the product',
    `sort_order`   INT(11)      DEFAULT 0 COMMENT 'Sort Order for display purposes',
    PRIMARY KEY (`product_id`) USING BTREE
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  ROW_FORMAT = DYNAMIC COMMENT = 'Product Information Table';
```

### Order Information Table

```sql
create table `tb_order_info`
(
    `order_id`         bigint(20) not null auto_increment comment 'Order ID',
    `user_id`          varchar(32)    default null comment 'User ID',
    `product_id`       bigint(20)     default null comment 'Product ID',
    `product_name`     varchar(100)   default null comment 'Product Name',
    `total_amount`     decimal(10, 2) default null comment 'Total Amount',
    `channel_order_id` varchar(50)    default null comment 'Channel Order ID',
    `status`           varchar(20)    default 'PENDING' comment 'Order Status',
    `create_time`      datetime       default null comment 'Order Creation Time',
    `payment_time`     datetime       default null comment 'Payment Time',
    `credits`          int(11)        default null comment 'Credits used for the order',
    `payment_info`     varchar(255)   default null comment 'Payment Information',
    `payment_method`   varchar(50)    default null comment 'Payment Method',
    primary key (`order_id`) using btree,
    key `idx_user_id` (`user_id`) using btree comment 'Index for quick lookups of orders by user',
    key `idx_product_id` (`product_id`) using btree comment 'Index for quick lookups of orders by product',
    key `idx_payment_method` (`payment_method`) using btree comment 'Index for quick lookups of orders by payment method',
    key `idx_payment_time` (`payment_time`) using btree comment 'Index for quick lookups of orders by payment time'
) engine = InnoDB
  default charset = utf8mb4
  row_format = DYNAMIC comment ='Order Information Table';
```

### Payment Code Information Table

```sql
create table `tb_payment_code_info`
(
    `payment_code` varchar(8)     NOT NULL COMMENT 'Payment Code',
    `user_id`      varchar(32)    NOT NULL COMMENT 'User ID associated with the payment code',
    `amount`       decimal(10, 2) NOT NULL COMMENT 'Amount associated with the payment code',
    `status`       varchar(20) DEFAULT 'Unused' COMMENT 'Status of the payment code (0: Unused, 1: Used)',
    `create_time`  datetime    default null comment 'Creation Time',
    `used_time`    datetime    default null comment 'Used Time',
    primary key (`payment_code`) using btree
) engine = InnoDB
  default charset = utf8mb4
  row_format = DYNAMIC comment = 'Payment Code Information Table';
```
