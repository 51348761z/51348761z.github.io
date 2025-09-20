---
title: MySQL Quickstart Guide
date: 2025-09-19 15:27:46 +0800
categories: [MySQL]
tags: [mysql]
excerpt: "A concise guide to MySQL covering data types, database and table management, data manipulation, advanced querying, transactions, indexes, and JDBC operations."
---

This guide provides a quick overview of MySQL, including its data types, database and table management, data manipulation, advanced querying techniques, transactions, indexes, and JDBC operations.

## What is MySQL?

MySQL is an open-source relational database management system (RDBMS) that uses Structured Query Language (SQL) for managing and manipulating databases. It is widely used for web applications and data storage due to its reliability, performance, and ease of use.

Here's a quickstart guide to get you up and running with MySQL.

### What is a Relational Database?

简单来说，**关系型数据库** (Relational Database) 是一种将数据存储在多个相互关联的**表 (Table)** 中的数据库。您可以把它想象成一个由很多结构化的、互相链接的 Excel 表格组成的集合。

它的核心思想和主要组成部分如下：

1.  **表 (Table)**: 数据被组织在一个或多个二维表中。每个表都包含**行 (Row)** 和 **列 (Column)**。例如，你可能有一个 `用户表` 和一个 `订单表`。

2.  **行 (Row)**: 也称为**记录 (Record)**，代表表中的一个实体。例如，`用户表` 中的每一行都代表一个具体的用户信息。

3.  **列 (Column)**: 也称为**字段 (Field)** 或**属性 (Attribute)**，描述了实体的一个特定方面。例如，`用户表` 可能包含 `用户ID`、`姓名`、`邮箱` 等列。

4.  **关系 (Relation)**: 这是“关系型”数据库的核心，指的是**表与表之间的联系**。这种联系是通过 **键 (Key)** 来建立和维护的。
    *   **主键 (Primary Key)**: 表中每一行的唯一标识符，比如 `用户表` 中的 `用户ID`。它的值不能重复，也不能为空。
    *   **外键 (Foreign Key)**: 一个表中的列，其值对应另一个表的主键。这是建立两个表之间联系的桥梁。

#### Example

假设我们有 `用户表 (Users)` 和 `订单表 (Orders)`。

**`用户表 (Users)`**

| 用户ID (主键) | 姓名 | 邮箱                 |
| :------------ | :--- | :------------------- |
| 101           | 张三 | zhangsan@example.com |
| 102           | 李四 | lisi@example.com     |

**`订单表 (Orders)`**

| 订单ID (主键) | 产品名称   | **用户ID (外键)** |
| :------------ | :--------- | :---------------- |
| 2001          | 笔记本电脑 | 101               |
| 2002          | 鼠标       | 102               |
| 2003          | 键盘       | 101               |

在这个例子中，`订单表` 中的 `用户ID` 列就是一个**外键**，它引用了 `用户表` 中的 `用户ID`（主键），从而建立了“哪个用户下了哪个订单”的**关系**。

#### Key Advantages

*   **减少数据冗余 (Reduced Data Redundancy)**：用户信息只存储在 `用户表` 中，无需在每个订单里都重复一遍。
*   **保证数据一致性和完整性 (Data Consistency and Integrity)**：由于有外键约束，你不能创建一个引用不存在用户的订单，保证了数据的准确性。
*   **查询灵活 (Flexible Querying)**：可以使用 **SQL (Structured Query Language)** 这种强大的语言，轻松地组合来自多个表的数据进行复杂查询。

> 常见的关系型数据库管理系统 (RDBMS) 包括 **MySQL**, **PostgreSQL**, **Microsoft SQL Server**, **Oracle Database** 等。
{: .prompt-info}

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
  show databases;
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
  select * from table_name where column_name like 'pattern';
  ```

  > `%` matches any sequence of characters; `_` matches a single character.
  {: .prompt-info}

## Advanced Querying

### Aggregate Functions

- Counts the total number of rows in a table:
  ```sql
  select count(*) from table_name;
  ```

- Counts the number of non-null values in a specific column:
  ```sql
  select count(column_name) from table_name;
  ```

- Calculates the average value of a numeric column:
  ```sql
  select avg(column_name) from table_name;
  ```

- Finds the maximum value in a column:
  ```sql
  select max(column_name) from table_name;
  ```

- Finds the minimum value in a column:
  ```sql
  select min(column_name) from table_name;
  ```

- Calculates the sum of all values in a numeric column:
  ```sql
  select sum(column_name) from table_name;
  ```

### Sorting Data (`ORDER BY`)

- Sorts the results by a specific column in ascending order (default):
  ```sql
  select * from table_name order by column_name;
  ```

- Sorts the results based on a condition and then by a column:
  ```sql
  select * from table_name where column_name = value order by column_name;
  ```

- Explicitly specifies ascending order for sorting:
  ```sql
  select * from table_name order by column_name asc;
  ```

- Specifies descending order for sorting:
  ```sql
  select * from table_name order by column_name desc;
  ```

- Sorts by multiple columns with different orders:
  ```sql
  select * from table_name order by column1 asc, column2 desc;
  ```

### Grouping Data (`GROUP BY`)

- Groups rows that have the same values in a specified column into summary rows. Often used with aggregate functions:
  ```sql
  select column_name, count(*) from table_name group by column_name;
  ```

### Limiting Results / Pagination (`LIMIT`)

- Retrieves a subset of rows from a query, useful for pagination. `start_position` is zero-based.
  ```sql
  select * from table_name limit start_position, number_of_rows;
  ```

### Joining Tables

Joins are used to combine rows from two or more tables based on a related column between them.

- **`INNER JOIN`**: Returns records that have matching values in both tables.
- **`LEFT JOIN`**: Returns all records from the left table, and the matched records from the right table.
- **`RIGHT JOIN`**: Returns all records from the right table, and the matched records from the left table.

The join condition is specified using `ON` or `WHERE`.

```sql
-- Using ON (recommended)
select * from table1
inner join table2 on table1.column_name = table2.column_name;

-- Using WHERE
select * from table1, table2
where table1.column_name = table2.column_name;
```

### Subqueries

A subquery is a query nested inside another query. The result of the inner query is used by the outer query.

- Subqueries can be used in `SELECT`, `FROM`, or `WHERE` clauses.

```sql
-- Subquery in a WHERE clause
select * from table1
where column1 in (select column1 from table2 where condition);
```

## Common Built-in Functions

### Date Functions
- `CURDATE()`, `CURRENT_DATE()`: Returns the current date.

### String Functions
- `CHAR_LENGTH(s)`: Returns the number of characters in string `s`.
- `LENGTH(s)`: Returns the length of string `s` in bytes.
- `CONCAT(s1, s2, ...)`: Concatenates multiple strings into one string.

### Math Functions
- `ABS(x)`: Returns the absolute value of `x`.
- `CEIL(x)`, `CEILING(x)`: Rounds `x` up to the nearest integer.
- `FLOOR(x)`: Rounds `x` down to the nearest integer.

### Aggregate Functions
- `AVG([DISTINCT] expr)`: Returns the average value of `expr`. The `DISTINCT` option ignores duplicate values.
- `COUNT([DISTINCT] expr)`: Returns a count of the number of non-NULL values of `expr`.
- `GROUP_CONCAT(...)`: Concaten  > `GROUP_CONCAT` 和 `CONCAT` 都是 MySQL 中用于字符串处理的函数，但用途不同：  
  > `CONCAT` 用于将多个字符串拼接成一个字符串。例如：`CONCAT(name, ':', email)` 会把 `name` 和 `email` 用冒号连接起来，结果是一条记录对应一个字符串。  
  > `GROUP_CONCAT` 用于将分组内的多个值合并成一个字符串，常用于聚合查询。例如：`GROUP_CONCAT(email)` 会把同一分组（如同分数）的所有邮箱地址用逗号拼接成一个字符串，结果是一组对应一个字符串。  
  > 简单来说，`CONCAT` 处理单行数据，`GROUP_CONCAT` 处理分组后的多行数据。  
  > `CONCAT` and `GROUP_CONCAT` are both string functions but serve different purposes:
    > - **`CONCAT`**: Joins strings from ***multiple columns*** within a single row.  
      > For example, `CONCAT(first_name, ' ', last_name)` combines the first and last name for each record. It operates horizontally on one row.
    > - **`GROUP_CONCAT`**: An aggregate function that joins strings from ***a single column across multiple rows*** within a group.  
      > For example, `GROUP_CONCAT(product_name)` would list all product names for a given category. It operates vertically on a group of rows.  
  >
  > In short: `CONCAT` is for single-row operations, while `GROUP_CONCAT` is for multi-row group operations.
  {: .prompt-tip}


## MySQL Transactions

在 MySQL 中，事务是一组SQL语句的执行，它们被视为一个单独的工作单元。一般来说，事务是必须满足4个条件（ACID）：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）。
> A transaction is a sequence of SQL statements executed as a single logical unit of work. Generally, a transaction must satisfy four conditions known as ACID: Atomicity, Consistency, Isolation, and Durability.
{: .prompt-info}

### Transaction Control

- `BEGIN` or `START TRANSACTION`: Starts a new transaction.
- `COMMIT`: Saves all changes made during the transaction, making them permanent.
- `ROLLBACK`: Reverts all changes made during the current transaction.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE user_id = 2;
COMMIT;
```

### Autocommit Mode

By default, MySQL runs in autocommit mode. You can change this using the `SET` command.
- `SET AUTOCOMMIT=0`: Disables autocommit mode.
- `SET AUTOCOMMIT=1`: Enables autocommit mode.

## MySQL Indexes

MySQL 索引是一种数据结构，用于加快数据库查询的速度和性能。索引的建立对于 MySQL 的高效运行是很重要的，它可以大大提高 MySQL 的检索速度。
> A MySQL index is a data structure used to improve the speed and performance of database queries. Creating appropriate indexes is crucial for the efficient operation of MySQL, as they can significantly speed up data retrieval.
{: .prompt-info}

### Creating an Index

You can create an index using `CREATE INDEX` or by altering the table.

```sql
CREATE INDEX index_name
ON table_name (column1, column2, ...);
```

```sql
ALTER TABLE table_name
ADD INDEX index_name (column1, column2, ...);
```

### Deleting an Index

You can delete an index using `DROP INDEX`.

```sql
DROP INDEX index_name ON table_name;
```

```sql
ALTER TABLE table_name
DROP INDEX index_name;
```

### Pros and Cons of Indexes

在数据库表中添加索引，主要会带来以下几个显著的不同。简单来说，索引就像一本书的目录。没有目录，你需要从头到尾翻阅整本书来找一个特定的章节（全表扫描）；有了目录，你可以直接找到章节所在的页码，快速定位（索引查找）。

#### Pros

1.  **大幅提升查询速度 (Faster `SELECT` Queries)**
    *   这是添加索引最主要的原因。对于带有 `WHERE` 子句的查询，数据库可以利用索引快速定位到符合条件的行，而无需逐行扫描整个表。
    *   对于 `JOIN` 操作，如果连接的列（通常是外键）上有索引，可以极大地提高连接效率。
    *   对于 `ORDER BY` 和 `GROUP BY` 操作，如果排序或分组的列上有索引，数据库可以直接按索引的顺序读取数据，避免了额外的排序开销。

2.  **保证数据唯一性 (Enforce Uniqueness)**
    *   通过创建唯一索引 (`UNIQUE INDEX`)，可以确保索引列中的所有值都是唯一的，这是一种强制数据完整性的有效手段。主键 (`PRIMARY KEY`) 本质上就是一个唯一的、非空的索引。

#### Cons

1.  **降低数据写入和修改的速度 (Slower `INSERT`, `UPDATE`, `DELETE` Operations)**
    *   当你向表中插入、更新或删除一行数据时，数据库不仅需要修改表中的数据，**还需要同时更新该表上的所有索引**，以确保索引与表数据保持同步。
    *   表上的索引越多，写操作（`INSERT`, `UPDATE`, `DELETE`）的开销就越大，速度也就越慢。

2.  **占用额外的磁盘空间 (Requires Additional Disk Space)**
    *   索引本身是一个独立的数据结构，需要存储在磁盘上。因此，每创建一个索引，都会消耗额外的存储空间。对于非常大的表，索引文件本身也可能变得非常大。

3.  **需要维护成本 (Maintenance Overhead)**
    *   数据库系统需要花费时间和资源来维护索引的正确性。随着数据的不断变化，索引可能会变得碎片化，需要进行重建或优化来保持其性能。

#### Summary Table

| 操作                | 无索引 (Before Index) | 有索引 (After Index)                    |
| :------------------ | :-------------------- | :-------------------------------------- |
| **查询 (`SELECT`)** | 慢 (需要全表扫描)     | **快** (通过索引快速定位)               |
| **写入 (`INSERT`)** | 快                    | **慢** (需要更新索引)                   |
| **更新 (`UPDATE`)** | 快                    | **慢** (如果更新了索引列，需要更新索引) |
| **删除 (`DELETE`)** | 快                    | **慢** (需要从索引中移除条目)           |
| **磁盘空间**        | 少                    | **多** (需要额外空间存储索引)           |

> **核心权衡**：索引是以**牺牲写入性能和占用更多空间**为代价，来换取**查询性能的巨大提升**。
> 因此，是否创建索引以及在哪些列上创建索引，需要根据应用的具体场景来决定。通常，对于读多写少的表，以及经常用于查询条件、连接、排序的列，创建索引是非常有益的。
{: .prompt-tip}

## JDBC Database Operations

Java Database Connectivity (JDBC) is an API for the Java programming language that defines how a client may access a database. It provides methods for querying and updating data in a database.

### Example: Establishing a Connection

Here is a basic Java code example that connects to a database using JDBC.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DatabaseConnection {
    // The database URL, typically in the format jdbc:mysql://hostname:port/database
    private static final String URL = "jdbc:mysql://localhost:3306/your_database_name";
    
    // Your database username
    private static final String USER = "your_username";
    
    // Your database password
    private static final String PASSWORD = "your_password";

    /**
     * Establishes a connection to the database.
     * @return A Connection object to the database.
     * @throws SQLException if a database access error occurs.
     */
    public static Connection getConnection() throws SQLException {
        // The DriverManager attempts to establish a connection to the given database URL.
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }
}
```

#### Code Explanation

1.  **`import` statements**: Imports the core JDBC classes from the `java.sql` package: `Connection`, `DriverManager`, and `SQLException`.
2.  **`URL`**: The database connection string. It specifies the database type (e.g., `mysql`), hostname (`localhost`), port (`3306`), and the database name. You need to replace this with your actual database information.
3.  **`USER` and `PASSWORD`**: The username and password for logging into the database.
4.  **`getConnection()` method**: This static method uses `DriverManager.getConnection()` to attempt to establish a database connection to the specified URL. If successful, it returns a `Connection` object, which represents the session with the database, and all subsequent SQL operations will be performed through this object. If the connection fails, it throws an `SQLException`.

## Troubleshooting

Here are some common issues you might encounter when working with MySQL and how to resolve them.

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

```shell
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

```shell
mysql -u root -p'your_code'
mysql: Deprecated program name. It will be removed in a future release, use '/usr/bin/mariadb' instead
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 12.0.2-MariaDB Arch Linux

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```
