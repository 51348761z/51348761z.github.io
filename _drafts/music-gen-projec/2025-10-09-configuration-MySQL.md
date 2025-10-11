---
title: 'Music-Gen Project: MySQL configuration'
date: 2025-10-09 17:44:54 +0800
categories: [Java, Project]
tags: [springboot, mysql, java]
mermaid: true
description: "This article discusses the configuration of MySQL for the Music-Gen project."
---

## Music-Gen Project: MySQL configuration

This article discusses the configuration of MySQL for the Music-Gen project.

### 1. Create Database and User Table

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

### 2. SQL Clause Explanations

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

### 3. Add Music Information Table

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
    PRIMARY KEY (`music_id`) USING BTREE,
    UNIQUE KEY `uk_task_id` (`task_id`) USING BTREE COMMENT 'Ensure task ID from the AI service is unique'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='Stores information about generated music tracks';
```

### 4. Java Data Type Mappings

When mapping SQL data types to Java data types, consider the following recommendations for optimal compatibility and performance:

| **SQL Data Type** | **Recommended Java Data Type** | **Reason**                                                                                        | **** |
|-------------------|--------------------------------|---------------------------------------------------------------------------------------------------|------|
| **INT**           | `Integer`                      | Precise 32-bit integer mapping, and can handle `NULL` values.                                     |      |
| **TINYINT(1)**    | `Integer` or `Boolean`         | `Integer` is more versatile; `Boolean` is semantically clearer if it's explicitly a boolean flag. |      |
| **BIGINT**        | `Long`                         | Precise 64-bit integer mapping, and can handle `NULL` values.                                     |      |
| **BINARY(16)**    | `java.util.UUID`               | Type-safe, semantically clear, and well-supported by frameworks.                                  |      |
| **VARCHAR**       | `String`                       | Standard string mapping.                                                                          |      |
| **DATETIME**      | `java.time.LocalDateTime`      | The standard way to handle date and time in modern Java.                                          |      |
