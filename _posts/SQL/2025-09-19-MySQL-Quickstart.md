---
title: MySQL quickstart
date: 2025-09-19 15:27:46 +0800
categories: [MySQL]
tags: [mysql]
---

## Data Types

### Numeric Data Types

|    **Type**    | **Size** |
| :------------: | :------: |
|    tinyint     | 1 Bytes  |
|    smallint    | 2 Bytes  |
|   mediumint    | 3 Bytes  |
| int OR integer | 4 Bytes  |
|     bigint     | 8 Bytes  |
|     float      | 4 Bytes  |
|     double     | 8 Bytes  |

### Date And Time Data Types

| **Data Type** | **Storage (Bytes)** | **Range**                                              | **Format**          | **Description**                                                                                  |
| ------------- | ------------------- | ------------------------------------------------------ | ------------------- | ------------------------------------------------------------------------------------------------ |
| date          | 3                   | '1000-01-01' to '9999-12-31'                           | YYYY-MM-DD          | Stores a date value (year, month, and day).                                                      |
| time          | 3                   | '-838:59:59' to '838:59:59'                            | HH:MM:SS            | Stores a time value or a duration between two times.                                             |
| year          | 1                   | 1901 to 2155                                           | YYYY                | Stores a year value.                                                                             |
| datetime      | 8                   | '1000-01-01 00:00:00' to '9999-12-31 23:59:59'         | YYYY-MM-DD HH:MM:SS | Stores a combination of a date and a time.                                                       |
| timestamp     | 4                   | '1970-01-01 00:00:01' UTC to '2038-01-19 03:14:07' UTC | YYYY-MM-DD HH:MM:SS | Stores a combination of date and time, timezone-aware (UTC). Often used to track record changes. |

> `timestamp` 只占 4 个字节，而且是以 UTC 的格式储存，会自动检索当前时区并进行转换。  
> `datetime` 以 8 个字节储存，不会进行时区的检索。  
> 如果存的是NULL，`timestamp` 会自动储存当前时间，而 `datetime` 会储存 NULL。
{: .prompt-tip}

### String Data Types

 | **Data Type** | **Maximum Size**           | **Description**           | **Usage Notes**                                                                                                        |
 | ------------- | -------------------------- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
 | char          | 255 bytes                  | Fixed-length string       | Always uses the same amount of storage, padding with spaces if needed. Fast for fixed-size data (e.g., country codes). |
 | varchar       | 65,535 bytes               | Variable-length string    | Only uses storage for the characters you enter. Good for most text fields (e.g., names, titles).                       |
 | tinytext      | 255 bytes                  | Short text string         | A very small text field.                                                                                               |
 | tinyblob      | 255 bytes                  | Short binary data         | For small binary data, like a tiny image or file.                                                                      |
 | text          | 65,535 bytes (64 KB)       | Long text data            | For long-form text like articles or descriptions. Has a character set.                                                 |
 | blob          | 65,535 bytes (64 KB)       | Binary data               | For binary data like small images or files. Has no character set.                                                      |
 | mediumtext    | 16,777,215 bytes (16 MB)   | Medium-length text        | For very long text, like the content of a book.                                                                        |
 | mediumblob    | 16,777,215 bytes (16 MB)   | Medium-length binary data | For medium-sized files, like videos or audio clips.                                                                    |
 | longtext      | 4,294,967,295 bytes (4 GB) | Extremely large text      | For huge text datasets.                                                                                                |

## Connect To The Database

```sql
> mysql -u'username' -p'password'
# or
> mysql -u'username' -p
```

## Database Management

- Lists all available databases on the server:

  ```sql
  show database;
  ```

- Creates a new, empty database with the specific name:

  ```sql
  create database database_name;
  ```

- Creates a new, empty database with a specific character set:

  ```sql
  create database database_name character set utf8mb4;
  ```

- Permanently deletes an existing database and all of its contents:

  ```sql
  drop database database_name;
  ```

## Table Management

- Switches the current session to the specified database:

  ```sql
  use database_name;
  ```

- Lists all tables in the currently selected database:

  ```sql
  show tables;
  ```

- Creates a new table with a defined set of columns and data types:

  ```sql
  create table_name (...);
  ```

  example:

  ```sql
  create table users (
    id int auto_increment primary key,
    username varchar(50) not null,
    email varchar(100) not null,
    birthday date,
    is_active boolean default true
  );
  ```

  > 使用 `boolean` 类型创建变量时，会自动将其存储为 `tinyint` 类型。
  {: .prompt-tip}

- Displays the structure of a table, including its columns and data types:

  ```sql
  desc table_name;
  ```

- Permanently deletes an existing table and all of its data:

  ```sql
  drop table table_name;
  ```

- Modifieds the structure of an existing table in various ways:

  ```sql
  alter table table_name ...;
  ```

  - Adds a new column to the table:

      ```sql
      ... add column column_name type;
      ```

  - Removes a column from the table:

      ```sql
      ... drop column column_name;
      ```

  - Changes the data type or attributes of an existing column:

      ```sql
      ... modify column column_name new_type;
      ```

  - Rename a column and can also change its data type:

      ```sql
      ... change column old_name new_name new_type;
      ```

## Data Manipulation (DML)

- Insert a new row of data into a table:

  ```sql
  insert into table_name (column1, column2) values (value1, value2);
  ```

- Deletes all rows from a table, but does not reset auto-increment counters:

  ```sql
  delete from table_name;
  ```

  - *A faster way to delete all rows and reset counters.*

    ```sql  
    truncate table table_name
    ```

- Deletes specific rows from a table that match the `where` condition:

  ```sql
  delete from table_name where condition;
  ```

- Modifies existing data in rows that match the `where` condition:

  ```sql
  update table_name set column1 = value1 where condition;
  ```

## Data Query (SELECT)

- Retrieves data from specific columns in a table:

  ```sql
  select column1, column2 from table_name;
  ```

- Retrieves data from a column and renames that column in the result set:
  
  ```sql
  select column_name as alias_name from table_name;
  ```

- Retrieves only the unique values from a specified column:
  
  ```sql
  select distinct column_name from table_name;
  ```

- Retrieves rows that match a specific condition in the `where` clause:
  
  ```sql
  select * from table_name where condition;
  ```

  > Common conditions use operators like `=`, `>`, `<`, `<>`, `and`, `or`, `is null`, `is not null`, `between`.
  {: .prompt-info}

- Combines string data from multiple columns into a single column:
  
  ```sql
  select concat(first_name, ' ', last_name) from table_name;
  ```

  > For numbers, you can use mathematical operators like `+`, `-`, `*`, `/` directly.
  {: .prompt-info}

- Searches for rows where a column's value matches a specified pattern:

  ```sql
  select * from table_name where column_name like 'patter';
  ```

  > `%` matches any sequence of characters; `_` matches a single character.
  {:. prompt-info}

## Troubleshootings

Some troubleshooting solutions.

### ERROR 1698 (28000): Access denied for user 'root'@'localhost'

> **Method: Change root Authentication Method**
>
> - Open up the MySQL prompt from your terminal:
>   1. sudo mysql;
>   2. SELECT user,plugin,host FROM mysql.user WHERE user = 'root';
>   3. ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'enter_password_here';
>   4. FLUSH PRIVILEGES;
>
> You should now be able to log in to phpMyAdmin using your root account.
{: .prompt-tip}

```sql
MariaDB [(none)]> select user, plugin, host from mysql.user where user='root';
+------+-----------------------+-----------+
| User | plugin                | Host      |
+------+-----------------------+-----------+
| root | mysql_native_password | localhost |
+------+-----------------------+-----------+
1 row in set (0.002 sec)

MariaDB [(none)]> alter user 'root'@'localhost' identified with mysql_native_password by '';
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'by '1'' at line 1

MariaDB [(none)]> alter user 'root'@'localhost' identified by 'your_code';
Query OK, 0 rows affected (0.038 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.001 sec)
```

then it's ok:

```sql
mysql -u root -p'your_code'
mysql: Deprecated program name. It will be removed in a future release, use '/usr/bin/mariadb' instead
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 12.0.2-MariaDB Arch Linux

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```
