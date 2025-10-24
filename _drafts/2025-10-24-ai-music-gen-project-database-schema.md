---
title: "AI Music Generation Project Database Schema"
date: 2025-10-24 14:09:29 +0800
categories: ["AI Music Generation", "Database Design"]
tags: ["AI", "Music Generation", "Database Schema", "Project Management"]
draft: true
---

Below is the proposed database schema for the AI Music Generation project. This schema includes tables for users, AI models, generation tasks, music works, and orders for point recharges.

```sql
-- Users Table: Stores user account information
CREATE TABLE `tb_users` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT 'Primary Key (Internal ID)',
  `uuid` VARCHAR(36) NOT NULL UNIQUE COMMENT 'Public unique identifier for API usage',
  `user_id_str` VARCHAR(50) NOT NULL UNIQUE COMMENT 'Unique string identifier assigned upon registration (immutable)',
  `email` VARCHAR(100) NOT NULL UNIQUE COMMENT 'User email address (used for login)',
  `password_hash` VARCHAR(255) NOT NULL COMMENT 'Hashed password',
  `nickname` VARCHAR(50) NOT NULL COMMENT 'User display name (mutable)',
  `avatar_url` VARCHAR(512) NULL COMMENT 'URL or path to the user avatar image',
  `role` VARCHAR(50) NOT NULL DEFAULT 'user' COMMENT 'User role identifier (e.g., user, admin)',
  `points_balance` INT NOT NULL DEFAULT 0 COMMENT 'User points balance for generating music',
  `status` VARCHAR(20) NOT NULL DEFAULT 'enabled' COMMENT 'Account status (enabled, disabled) - Maps to CodeEnum<String>',
  `version` INT NOT NULL DEFAULT 0 COMMENT 'Optimistic locking version number',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'Record creation timestamp',
  `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Record last update timestamp',
  PRIMARY KEY (`id`),
  INDEX `idx_uuid` (`uuid`),
  INDEX `idx_email` (`email`),
  INDEX `idx_user_id_str` (`user_id_str`)
) COMMENT='Stores user account information';

-- AI Models Table: Stores available AI music generation models
CREATE TABLE `tb_ai_models` (
  `id` INT NOT NULL AUTO_INCREMENT COMMENT 'Primary Key',
  `model_key` VARCHAR(50) NOT NULL UNIQUE COMMENT 'Unique key identifier for the model (e.g., basic_v1)',
  `model_name` VARCHAR(100) NOT NULL COMMENT 'Display name of the model',
  `description` TEXT NULL COMMENT 'Description of the model',
  `points_cost` INT NOT NULL DEFAULT 1 COMMENT 'Points required to use this model',
  `status` VARCHAR(20) NOT NULL DEFAULT 'active' COMMENT 'Model status (active, inactive) - Maps to CodeEnum<String>',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'Record creation timestamp',
  `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Record last update timestamp',
  PRIMARY KEY (`id`),
  INDEX `idx_model_key` (`model_key`)
) COMMENT='Stores available AI music generation models';

-- Generation Tasks Table: Tracks asynchronous music generation tasks
CREATE TABLE `tb_generation_tasks` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT 'Primary Key (Task ID)',
  `user_id` BIGINT NOT NULL COMMENT 'Foreign key referencing the user who initiated the task',
  `ai_model_id` INT NOT NULL COMMENT 'Foreign key referencing the AI model used', 
  `prompt` TEXT NOT NULL COMMENT 'Text prompt provided by the user',
  `status` VARCHAR(20) NOT NULL DEFAULT 'PENDING' COMMENT 'Task status (PENDING, PROCESSING, COMPLETED, FAILED) - Maps to CodeEnum<String>',
  `error_message` TEXT NULL COMMENT 'Error message if the task failed',
  `result_url` VARCHAR(512) NULL COMMENT 'URL of the generated music file upon success',
  `duration_seconds` INT NULL COMMENT 'Duration of the generated music in seconds',
  `points_consumed` INT NOT NULL COMMENT 'Points consumed for this task',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'Record creation timestamp',
  `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Record last update timestamp',
  PRIMARY KEY (`id`),
  INDEX `idx_user_id` (`user_id`),
  INDEX `idx_status` (`status`),
  FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE CASCADE,
  FOREIGN KEY (`ai_model_id`) REFERENCES `ai_models`(`id`) 
) COMMENT='Tracks asynchronous music generation tasks';

-- Music Works Table: Stores user-generated music pieces
CREATE TABLE `tb_music_works` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT 'Primary Key',
  `uuid` VARCHAR(36) NOT NULL UNIQUE COMMENT 'Public unique identifier for the music work',
  `generation_task_id` BIGINT NOT NULL UNIQUE COMMENT 'Foreign key referencing the generation task that created this work', 
  `user_id` BIGINT NOT NULL COMMENT 'Foreign key referencing the owner user',
  `title` VARCHAR(255) NULL DEFAULT 'Untitled Work' COMMENT 'Title of the music work (mutable by user)',
  `file_url` VARCHAR(512) NOT NULL COMMENT 'URL of the music file (potentially redundant, but useful for quick access)',
  `duration_seconds` INT NULL COMMENT 'Duration of the music in seconds (potentially redundant)',
  `visibility` VARCHAR(10) NOT NULL DEFAULT 'private' COMMENT 'Visibility status (private, public) - Maps to CodeEnum<String>',
  `publish_time` DATETIME NULL COMMENT 'Timestamp when the work was made public',
  `play_count` INT NOT NULL DEFAULT 0 COMMENT 'Number of times the work has been played',
  `like_count` INT NOT NULL DEFAULT 0 COMMENT 'Number of likes the work has received',
  `is_featured` TINYINT NOT NULL DEFAULT 0 COMMENT 'Flag indicating if the work is featured (0: No, 1: Yes)',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'Record creation timestamp',
  `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Record last update timestamp',
  PRIMARY KEY (`id`),
  INDEX `idx_uuid` (`uuid`),
  INDEX `idx_user_id` (`user_id`),
  INDEX `idx_visibility` (`visibility`),
  INDEX `idx_publish_time` (`publish_time`),
  FOREIGN KEY (`generation_task_id`) REFERENCES `generation_tasks`(`id`) ON DELETE CASCADE,
  FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE CASCADE
) COMMENT='Stores user-generated music pieces';

-- Orders Table: Stores payment orders for point recharges (Future Extension)
CREATE TABLE `tb_orders` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT 'Primary Key',
  `order_no` VARCHAR(64) NOT NULL UNIQUE COMMENT 'Unique order number',
  `user_id` BIGINT NOT NULL COMMENT 'Foreign key referencing the user making the purchase',
  `product_desc` VARCHAR(255) NOT NULL COMMENT 'Description of the purchased item (e.g., Recharge 100 points)',
  `amount_cny` DECIMAL(10, 2) NOT NULL COMMENT 'Payment amount in CNY',
  `points_recharged` INT NOT NULL COMMENT 'Number of points recharged',
  `status` VARCHAR(20) NOT NULL DEFAULT 'UNPAID' COMMENT 'Order status (UNPAID, PAID, CANCELLED) - Maps to CodeEnum<String>',
  `payment_gateway` VARCHAR(20) NULL COMMENT 'Payment gateway used (e.g., wechat, alipay)',
  `paid_time` DATETIME NULL COMMENT 'Timestamp when the order was paid',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'Record creation timestamp',
  `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Record last update timestamp',
  PRIMARY KEY (`id`),
  INDEX `idx_order_no` (`order_no`),
  INDEX `idx_user_id` (`user_id`),
  INDEX `idx_status` (`status`),
  FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE SET NULL -- Set user_id to NULL if user is deleted, to keep order history
) COMMENT='Stores payment orders for point recharges';
```
