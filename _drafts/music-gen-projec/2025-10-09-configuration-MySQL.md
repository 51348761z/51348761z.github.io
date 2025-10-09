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

### 1. Create Database and User

```sql
CREATE DATABASE music_gen_db;
USE music_gen_db;
CREATE TABLE `user_info` (
  `user_id` varchar(12) NOT NULL COMMENT 'User ID',
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

-   **What it is**: `BTREE` stands for B-Tree, which is the default and most common index data structure in MySQL's InnoDB storage engine.
-   **Why it's used**: B-Tree indexes are highly efficient for a wide variety of queries. They store data in a sorted order, which makes them excellent for:
    -   **Exact Lookups**: Quickly finding a row by its exact value (e.g., `WHERE user_id = 'some-id'`). This is essential for primary keys.
    -   **Range Queries**: Efficiently retrieving rows within a range (e.g., `WHERE create_time > '2025-01-01'`).
-   **Is it necessary?**: No. Since `BTREE` is the default for InnoDB, explicitly stating `USING BTREE` is optional. The database would use it anyway. However, it can be useful for code clarity.

#### What is `ROW_FORMAT=DYNAMIC`?

This setting defines how MySQL stores row data on disk. `DYNAMIC` is the modern default for the InnoDB engine and is generally the best choice.

-   **What it does**: It optimizes storage for variable-length columns (like `VARCHAR` or `TEXT`).
-   **Why it's used**: Instead of trying to cram the beginning of large text or blob data into the index page itself (like the older `COMPACT` format did), `DYNAMIC` stores the entire large column in separate "overflow pages." This keeps the primary index pages lean and fast, improving overall query performance because more index entries can fit into memory.
-   **Is it necessary?**: In modern MySQL (5.7+), `DYNAMIC` is the default. Specifying it is good practice for ensuring consistent behavior across different server configurations.
