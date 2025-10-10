---
title: Mybatis-plus Quickstart
date: 2025-09-27 10:00:00 +0800
categories: [Java, Mybatis-plus]
tags: [java, mybatis-plus, mysql]
description: Mybatis-plus quickstart guide.
---

This is a quickstart guide for Mybatis-plus, a powerful enhancement to MyBatis that simplifies CRUD operations and provides additional features.

## 1. Add Dependency

Add the following dependency to your `pom.xml` if you're using Maven:

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.10</version>
</dependency> 
```

## 2. Configure Data Source

Configure your data source in `application.properties` or `application.yml`:

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/your_database
    username: your_username
    password: your_password
    driver-class-name: com.mysql.cj.jdbc.Driver
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # Enable SQL logging
```

---

## 3. Quick Start

### 3.1 Create data table

```sql
create table user (
  id bigint auto_increment not null comment 'Primary Key',
  name varchar(50) character set utf8mb4 collate utf8mb4_general_ci default null comment 'User Name',
  age int default null comment 'User Age',
  email varchar(50) character set utf8mb4 collate utf8mb4_general_ci default null comment 'User Email',
  version bigint not null default '0' comment 'Version Number for Optimistic Locking',
  create_time datetime not null default current_timestamp comment 'Creation Time',
  update_time datetime not null default current_timestamp on update current_timestamp comment 'Update Time',
  primary key (id)
) engine=InnoDB default charset=utf8mb4 collate=utf8mb4_general_ci comment='User Table';
```

#### Table Creation Notes

- `character set` 指定字符编码（如 `utf8mb4` 支持完整 Unicode），确保存储与读取一致。
- `collate` 指定排序与比较规则（如 `utf8mb4_general_ci` 为不区分大小写的通用排序）。
- `TIMESTAMP`、`DATETIME` 通常映射到 `java.time.LocalDateTime`（旧版本可用 `java.sql.Timestamp`）。
- `CURRENT_TIMESTAMP` 等标识符（含 `NOW()`、`LOCALTIME`、`LOCALTIMESTAMP`）可用于 `DEFAULT` 或 `ON UPDATE` 获取当前时间。  
  - `CURRENT_TIMESTAMP`：MySQL 标准，支持 `DEFAULT` 和 `ON UPDATE`。
  - `NOW()`：MySQL 专用，等同于 `CURRENT_TIMESTAMP`，但不支持 `ON UPDATE`。
  - `LOCALTIME` 和 `LOCALTIMESTAMP`：SQL 标准，等同于 `CURRENT_TIMESTAMP`，但不支持 `ON UPDATE`。
  - `ON UPDATE` 只能用于 `TIMESTAMP` 和 `DATETIME` 列。
- `DEFAULT CURRENT_TIMESTAMP` 插入时自动赋值，`ON UPDATE CURRENT_TIMESTAMP` 更新行时刷新；可组合使用。
- `ENGINE=InnoDB` 指定存储引擎，常见选项还有 `MyISAM`、`MEMORY`、`CSV`、`ARCHIVE`、`BLACKHOLE` 及插件引擎。
  - `InnoDB`：支持事务、行级锁，适合**高并发读写**。
  - `MyISAM`：不支持事务，适合**读多写少**的场景。
  - `MEMORY`：数据存储在内存中，适合**临时数据处理**。
  - `CSV`：以 CSV 格式存储数据，适合**数据交换**。
  - `ARCHIVE`：适合存储大量**历史数据**，压缩存储。
  - `BLACKHOLE`：数据写入后丢弃，适合**日志收集**。

### 3.2 Create Entity Class

`com.wongs.mybatisplusdemo.entity.User`{: .filepath}

```java
package com.wongs.mybatisplusdemo.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;
import java.time.LocalTime;

@Data
@TableName(value = "user")
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class User {

    @TableId(type = IdType.AUTO)
    private Long id;

    private String name;

    private Integer age;

    private String email;

    @TableField(value = "create_time", fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    @TableField(value = "update_time", fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;

    @Version
    private Long version;

}
```

#### Entity Class Annotation Summary

- `@TableName("user")`: maps the entity to the user table.
- `@TableId(type = IdType.AUTO)`: marks id as the primary key using auto-increment strategy.
- `@TableField(value = "create_time", fill = FieldFill.INSERT)`: binds `createTime` to `create_time` and auto-fills on insert.
- `@TableField(value = "update_time", fill = FieldFill.INSERT_UPDATE)`: maps `updateTime` to `update_time` and auto-fills on insert/update.
- `@Version`: enables optimistic locking via the version column.

> `IdType.AUTO`：使用数据库自增 `ID` 作为主键。  
> `IdType.NONE`：无特定生成策略，如果全局配置中有 `IdType` 相关的配置，则会跟随全局配置。
> `IdType.INPUT`：在插入数据前，由用户自行设置主键值。
> `IdType.ASSIGN_ID`：自动分配 `ID`，适用于 `Long`、`Integer`、`String` 类型的主键。默认使用雪花算法通过 `IdentifierGenerator` 的 `nextId` 实现。
> `IdType.ASSIGN_UUID`：自动分配 `UUID`，适用于 `String` 类型的主键。默认使用 `UUID` 生成器通过 `IdentifierGenerator` 的 `nextUUID` 实现。
{: .prompt-info }

### 3.3 Create Mapper Interface

`com.wongs.mybatisplusdemo.mapper.UserMapper`{: .filepath}

```java
package com.wongs.mybatisplusdemo.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.wongs.mybatisplusdemo.entity.User;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface UserMapper extends BaseMapper<User> {
  // No additional methods needed; BaseMapper provides CRUD operations
}
```

#### Mapper Interface Summary

BaseMapper 是 Mybatis-Plus 提供的一个通用 Mapper 接口，它封装了一系列常用的数据库操作方法，包括增、删、改、查等。通过继承 BaseMapper，开发者可以快速地对数据库进行操作，而无需编写繁琐的 SQL 语句。

- `extends BaseMapper<User>`: inherits CRUD methods for User entity.
- `@Mapper`: marks the interface as a MyBatis mapper for component scanning.
- No need for XML mapper files; MyBatis-plus generates SQL based on method names and entity mappings.

### 3.4 Add `@MapperScan` to main application class

`com.wongs.mybatisplusdemo.MybatisPlusDemoApplication`{: .filepath}

```java
package com.wongs.mybatisplusdemo;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("com.wongs.mybatisplusdemo.mapper")
public class MybatisPlusDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusDemoApplication.class, args);
    }

}
```

#### `@MapperScan` Summary

- `@MapperScan("com.wongs.mybatisplusdemo.mapper")`: specifies the package to scan for mapper interfaces, enabling MyBatis to find and register them.

---

## 4. Common Usage Examples

### 4.1 CRUD Operations

MyBatis-plus provides built-in methods for common CRUD operations including `insert` (create), `selectById` (read), `updateById` (update), and `deleteById` (delete), etc.

#### Create

```java
@Test
void testInsert() {
    User user = User.builder()
        .name("John Doe")
        .age(30)
        .email("john.doe@example.com")
        .build();
    userMapper.insert(user);
}
```

```sql
-- SQL settings for create_time and update_time
alter table user modify column create_time datetime default current_timestamp comment 'Creation Time';
alter table user modify column update_time datetime default current_timestamp on update current_timestamp comment 'Update Time';
```

```text
JDBC Connection [HikariProxyConnection@168702939 wrapping com.mysql.cj.jdbc.ConnectionImpl@5d9d8e46] will not be managed by Spring
==>  Preparing: INSERT INTO user ( name, age, email, create_time, update_time ) VALUES ( ?, ?, ?, ?, ? )
==> Parameters: robot(String), 20(Integer), xxx@abc.com(String), null, null
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@779228dc]
```

> 当数据库允许传入 `null` 字段，并且 `createTime`、`updateTime` 未赋值时，MyBatis-plus 会写入 `null`，数据库默认值不会生效，这与我们的预期不符。
{: .prompt-danger }

##### 方案 1：仅在有值时写入列

```java
public class User {
    // ...other fields...

    @TableField(value = "create_time", insertStrategy = FieldStrategy.NOT_NULL)
    private LocalDateTime createTime;

    @TableField(value = "update_time", fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
    // ...other fields...
}
```

通过 `insertStrategy = FieldStrategy.NOT_NULL`，空值时跳过该列，保留数据库默认值（启用该策略后不要再设置 `fill`，因为 `fill` 会使得 `insertStrategy` 的配置被忽略，即断言该字段必有值）。

##### 方案 2：数据库强制非空并手动赋值

```sql
alter table user modify column create_time datetime not null default current_timestamp comment 'Creation Time';
alter table user modify column update_time datetime not null default current_timestamp on update current_timestamp comment 'Update Time';
```

```java
@Test
void testInsert() {
    LocalDateTime now = LocalDateTime.now();
    User user = User.builder()
        .name("John Doe")
        .age(30)
        .email("john.doe@example.com")
        .createTime(now)
        .updateTime(now)
        .build();
    userMapper.insert(user);
}
```

对列加非空约束并在代码中显式赋值，可确保时间戳始终符合业务要求。

#### Read

```java
@Test
void testRead() {
    User user = userMapper.selectById(5L);
    System.out.println(user);
}
```

```text
JDBC Connection [HikariProxyConnection@1535684464 wrapping com.mysql.cj.jdbc.ConnectionImpl@546083d6] will not be managed by Spring
==>  Preparing: SELECT id,name,age,email,create_time,update_time,version FROM user WHERE id=?
==> Parameters: 5(Long)
<==    Columns: id, name, age, email, create_time, update_time, version
<==        Row: 5, Eve, 50, eve@example.com, 2025-09-28 05:05:23, 2025-09-28 07:13:20, 0
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@68f776ee]
User(id=5, name=Eve, age=50, email=eve@example.com, createTime=2025-09-28T05:05:23, updateTime=2025-09-28T07:13:20, version=0)
```

#### Update

```java
@Test
void testUpdate(){
    // method 1, 根据实体entity和条件构造器wrapper进行更新
    User user = User.builder()
            .age(52)
            .updateTime(LocalDateTime.now())
            .build();
    UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
    updateWrapper.eq("id", 1L);
    userMapper.update(user, updateWrapper);

    // method 2
    // 根据入参entity的id（主键）进行更新，对于entity中非空的属性，会出现在UPDATE语句的SET后面，即entity中非空的属性，会被更新到数据库中。
    User user1 = User.builder()
            .id(1L)
            .age(30)
            .updateTime(LocalDateTime.now())
            .build();
    userMapper.updateById(user1);
}
```

```text
JDBC Connection [HikariProxyConnection@170132562 wrapping com.mysql.cj.jdbc.ConnectionImpl@6807989e] will not be managed by Spring
==>  Preparing: UPDATE user SET age=?, update_time=? WHERE (id = ?)
==> Parameters: 52(Integer), 2025-09-28T07:31:20.648017(LocalDateTime), 1(Long)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7e50eeb9]
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7a274521] was not registered for synchronization because synchronization is not active
JDBC Connection [HikariProxyConnection@799306600 wrapping com.mysql.cj.jdbc.ConnectionImpl@6807989e] will not be managed by Spring
==>  Preparing: UPDATE user SET age=?, update_time=? WHERE id=?
==> Parameters: 30(Integer), 2025-09-28T07:31:20.857118(LocalDateTime), 1(Long)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7a274521]
```

#### Delete

`BaseMapper` 一共提供了如下几个用于删除的方法

- `deleteById` 根据主键 `id` 进行删除
- `deleteBatchIds` 根据主键 `id` 进行批量删除
- `deleteByMap` 根据 `Map` 进行删除（`Map` 中的 `key` 为列名，`value` 为值，根据列和值进行等值匹配）
- `delete(Wrapper wrapper)` 根据条件构造器 `Wrapper` 进行删除

```java
@Test
void testDelete() {
    userMapper.deleteById(50L);
}
```

```text
JDBC Connection [HikariProxyConnection@288398804 wrapping com.mysql.cj.jdbc.ConnectionImpl@18ac4af6] will not be managed by Spring
==>  Preparing: DELETE FROM user WHERE id=?
==> Parameters: 50(Long)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7a3643e3]
```

### 4.2 Wrapper - 条件构造器

MyBatis-Plus 提供了一套强大的条件构造器（Wrapper），用于构建复杂的数据库查询条件。Wrapper 类允许开发者以链式调用的方式构造查询条件，无需编写繁琐的 SQL 语句，从而提高开发效率并减少 SQL 注入的风险。
> SQL注入是一种攻击手法，攻击者通过在输入中插入恶意SQL语句来操控数据库查询。若应用直接拼接未校验的输入，攻击者可绕过认证、窃取或篡改数据。使用参数化查询和输入校验可有效防范。
{: .prompt-info }

在 MyBatis-Plus 中，`Wrapper` 类是构建查询和更新条件的核心工具。以下是主要的 `Wrapper` 类及其功能：

- **AbstractWrapper**：这是一个抽象基类，提供了所有 `Wrapper` 类共有的方法和属性。它定义了条件构造的基本逻辑，包括字段（`column`）、值（`value`）、操作符（`condition`）等。所有的 `QueryWrapper`、`UpdateWrapper`、`LambdaQueryWrapper` 和 `LambdaUpdateWrapper` 都继承自 `AbstractWrapper`。

- **QueryWrapper**：专门用于构造查询条件，支持基本的等于、不等于、大于、小于等各种常见操作。它允许你以链式调用的方式添加多个查询条件，并且可以组合使用 `and` 和 `or` 逻辑。

- **UpdateWrapper**：用于构造更新条件，可以在更新数据时指定条件。与 `QueryWrapper` 类似，它也支持链式调用和逻辑组合。使用 `UpdateWrapper` 可以在不创建实体对象的情况下，直接设置更新字段和条件。

- **LambdaQueryWrapper**：这是一个基于 Lambda 表达式的查询条件构造器，它通过 Lambda 表达式来引用实体类的属性，从而避免了硬编码字段名。这种方式提高了代码的可读性和可维护性，尤其是在字段名可能发生变化的情况下。

- **LambdaUpdateWrapper**：类似于 `LambdaQueryWrapper`，`LambdaUpdateWrapper` 是基于 Lambda 表达式的更新条件构造器。它允许你使用 Lambda 表达式来指定更新字段和条件，同样避免了硬编码字段名的问题。

`AbstractWrapper` 中用于构建SQL语句中的 `WHERE` 条件的方法进行部分列举:

- `eq`：equals，等于
- `allEq`：all equals，全等于
- `ne`：not equals，不等于 `!=`
- `gt`：greater than ，大于 `>`
- `ge`：greater than or equals，大于等于 `≥`
- `lt`：less than，小于 `<`
- `le`：less than or equals，小于等于 `≤`
- `between`：相当于SQL中的 `BETWEEN`
- `notBetween`
- `like`：模糊匹配。`like("name","黄")`，相当于SQL的 `name like '%黄%'`
- `likeRight`：模糊匹配右半边。`likeRight("name","黄")`，相当于SQL的 `name like '黄%'`
- `likeLeft`：模糊匹配左半边。`likeLeft("name","黄")`，相当于SQL的 `name like '%黄'`
- `notLike`：`notLike("name","黄")`，相当于SQL的 `name not like '%黄%'`
- `isNull`：`isNull("name")`，相当于SQL的 `name is null`
- `isNotNull`：`isNotNull("name")`，相当于SQL的 `name is not null`
- `in`：`in("id", 1, 2, 3)`，相当于SQL的 `id in (1, 2, 3)`
- `and`：SQL连接符 `AND`
- `or`：SQL连接符 `OR`
- `apply`：用于拼接SQL，该方法可用于数据库函数，并可以动态传参

#### Conditional Query Example

##### (1) QueryWrapper and UpdateWrapper

```java
@Test
void testQueryAndUpdateWrapper(){
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    // conditional query
    queryWrapper.gt("age", 50);
    List<User> userList = userMapper.selectList(queryWrapper);

    // conditional update
    UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
    updateWrapper.eq("name", "David").set("age", 18);
    userMapper.update(updateWrapper);

}
```

```text
JDBC Connection [HikariProxyConnection@753027610 wrapping com.mysql.cj.jdbc.ConnectionImpl@46ee7013] will not be managed by Spring
==>  Preparing: SELECT id,name,age,email,create_time,update_time,version FROM user WHERE (age > ?)
==> Parameters: 50(Integer)
<==    Columns: id, name, age, email, create_time, update_time, version
<==        Row: 68, Nancy, 55, nancy@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 69, Oscar, 60, oscar@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 78, Xavier, 52, xavier@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 86, Felix, 53, felix@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 90, Jack, 58, jack@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 94, Nick, 51, nick@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 99, Sara, 54, sara@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==      Total: 7
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7858d31d]
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@22ff11ef] was not registered for synchronization because synchronization is not active
JDBC Connection [HikariProxyConnection@1613378103 wrapping com.mysql.cj.jdbc.ConnectionImpl@46ee7013] will not be managed by Spring
==>  Preparing: UPDATE user SET age=? WHERE (name = ?)
==> Parameters: 18(Integer), David(String)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@22ff11ef]
```

##### (2) Condition

条件构造器的诸多方法中，均可以指定一个boolean类型的参数condition，用来决定该条件是否加入最后生成的WHERE语句中，比如:

```java
@Test
void testCondition(){
    String name = "m";
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like(StringUtils.hasText(name), "name", name);
    List<User> users = userMapper.selectList(queryWrapper);
    // equivalent to
    if (StringUtils.hasText(name)) {
        queryWrapper.like("name", name);
        List<User> users2 = userMapper.selectList(queryWrapper);
    }
}
```

```text
JDBC Connection [HikariProxyConnection@753027610 wrapping com.mysql.cj.jdbc.ConnectionImpl@46ee7013] will not be managed by Spring
==>  Preparing: SELECT id,name,age,email,create_time,update_time,version FROM user WHERE (name LIKE ?)
==> Parameters: %m%(String)
<==    Columns: id, name, age, email, create_time, update_time, version
<==        Row: 67, Mike, 29, mike@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 81, Amy, 32, amy@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 85, Emily, 36, emily@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 93, Mary, 33, mary@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 100, Tom, 30, tom@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==      Total: 5
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@53ba7997]
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@5e055ce1] was not registered for synchronization because synchronization is not active
JDBC Connection [HikariProxyConnection@846778469 wrapping com.mysql.cj.jdbc.ConnectionImpl@46ee7013] will not be managed by Spring
==>  Preparing: SELECT id,name,age,email,create_time,update_time,version FROM user WHERE (name LIKE ? AND name LIKE ?)
==> Parameters: %m%(String), %m%(String)
<==    Columns: id, name, age, email, create_time, update_time, version
<==        Row: 67, Mike, 29, mike@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 81, Amy, 32, amy@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 85, Emily, 36, emily@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 93, Mary, 33, mary@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 100, Tom, 30, tom@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==      Total: 5
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@5e055ce1]
```

##### (3) Entity as Condition

调用构造函数创建一个Wrapper对象时，可以传入一个实体对象。后续使用这个Wrapper时，会以实体对象中的非空属性，构建WHERE条件（默认构建等值匹配的WHERE条件，这个行为可以通过实体类里各个字段上的@TableField注解中的condition属性进行改变）。

```java
@Test
void textEntityAsCondition(){
    User user = User.builder()
            .name("Tom")
            .build();
    QueryWrapper<User> userQueryWrapper = new QueryWrapper<>(user);
    List<User> users = userMapper.selectList(userQueryWrapper);
}
```

```text
JDBC Connection [HikariProxyConnection@1875372072 wrapping com.mysql.cj.jdbc.ConnectionImpl@49925d21] will not be managed by Spring
==>  Preparing: SELECT id,name,age,email,create_time,update_time,version FROM user WHERE name=?
==> Parameters: Tom(String)
<==    Columns: id, name, age, email, create_time, update_time, version
<==        Row: 100, Tom, 30, tom@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@338d47b]
```

##### (4) LambdaWrapper

lambda条件构造器，支持lambda表达式，可以不必像普通条件构造器一样，以字符串形式指定列名，它可以直接以实体类的方法引用来指定列。

像普通的条件构造器，列名是用字符串的形式指定，无法在编译期进行列名合法性的检查，这就不如lambda条件构造器来的优雅。

```java
@Test
void testLambdaQuery(){
    LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
    lambdaQueryWrapper.like(User::getName, "T").lt(User::getAge, 50);
    List<User> users = userMapper.selectList(lambdaQueryWrapper);
}
```

```text
JDBC Connection [HikariProxyConnection@1186076210 wrapping com.mysql.cj.jdbc.ConnectionImpl@7404aff2] will not be managed by Spring
==>  Preparing: SELECT id,name,age,email,create_time,update_time,version FROM user WHERE (name LIKE ? AND age < ?)
==> Parameters: %T%(String), 50(Integer)
<==    Columns: id, name, age, email, create_time, update_time, version
<==        Row: 71, Quentin, 30, quentin@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 73, Steve, 42, steve@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 74, Tina, 26, tina@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 76, Victor, 37, victor@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 83, Cathy, 24, cathy@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 91, Kate, 22, kate@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 100, Tom, 30, tom@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==      Total: 7
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@dae5e0]
```

##### (5) Custom SQL

- Annotation

  ```java
  @Select("select * from user)
  List<User> selectAllUsers();
  ```

- XML Mapper

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.wongs.mybatisplusdemo.mappers.UserMapper">
    <select id="selectRaw" resultType="com.wongs.mybatisplusdemo.po.User">
          SELECT * FROM user
      </select>
  </mapper>


  public interface UserMapper extends BaseMapper<User> {
    List<User> selectRaw();
  }
  ```

##### (6) Pagination

MyBatis-Plus 提供了内置的分页功能，简化了分页查询的实现。
> 分页是将大量数据分割成多个页面进行显示的技术，常用于Web应用中以提升用户体验和性能。通过分页，用户可以逐页浏览数据，而不是一次性加载所有数据，从而减少加载时间和服务器压力。
{: .prompt-info }

配置分页插件，在 'MybatisPlusConfig' 类中添加分页插件配置。
> MyBatis-Plus 的分页插件 `PaginationInnerInterceptor` 提供了强大的分页功能，支持多种数据库，使得分页查询变得简单高效。  
> 于 `v3.5.9` 起，`PaginationInnerInterceptor` 已分离出来。如需使用，则需单独引入 mybatis-plus-jsqlparser 依赖. 具体配置请参考 [MyBatis-Plus 官方文档](https://baomidou.com/getting-started/install/)。
{: .prompt-info }

`com.wongs.mybatisplusdemo.config.MybatisPlusConfig`{: .filepath}

```java
@Configuration
public class MyBatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor (){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL)); // 如果配置多个插件, 切记分页最后添加
        // 如果有多数据源可以不配具体类型, 否则都建议配上具体的 DbType
        return interceptor;
    }
}
```

```java
@Test
void testPagination() {
    LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
    lambdaQueryWrapper.between(User::getAge, 20, 50);

    Page<User> page = new Page<>(3, 2); // query page 3, 2 records per page
    Page<User> userPage = userMapper.selectPage(page, lambdaQueryWrapper); // execute pagination query
    System.out.println("Total records: " + userPage.getTotal());
    System.out.println("Total pages: " + userPage.getPages());
    System.out.println("Current page records: " + userPage.getCurrent());
    List<User> records = userPage.getRecords(); // Get current page records
    records.forEach(System.out::println);
}
```

```text
JDBC Connection [HikariProxyConnection@1359003971 wrapping com.mysql.cj.jdbc.ConnectionImpl@7a021f49] will not be managed by Spring
==>  Preparing: SELECT COUNT(*) AS total FROM user WHERE (age BETWEEN ? AND ?)
==> Parameters: 20(Integer), 50(Integer)
<==    Columns: total
<==        Row: 42
<==      Total: 1
==>  Preparing: SELECT id,name,age,email,create_time,update_time,version FROM user WHERE (age BETWEEN ? AND ?) LIMIT ?,?
==> Parameters: 20(Integer), 50(Integer), 4(Long), 2(Long)
<==    Columns: id, name, age, email, create_time, update_time, version
<==        Row: 60, Frank, 50, frank@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==        Row: 62, Heidi, 27, heidi@example.com, 2025-09-28 08:58:40, 2025-09-28 08:58:40, 0
<==      Total: 2
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@225e09f0]
Total records: 42
Total pages: 21
Current page records: 3
User(id=60, name=Frank, age=50, email=frank@example.com, createTime=2025-09-28T08:58:40, updateTime=2025-09-28T08:58:40, version=0)
User(id=62, name=Heidi, age=27, email=heidi@example.com, createTime=2025-09-28T08:58:40, updateTime=2025-09-28T08:58:40, version=0)
```

##### (7) Code Generator

MyBatis Plus 提供了代码生成器，可以根据数据库表自动生成实体类、Mapper 接口、Service 类和 Controller 类。
![MyBatis-Plus 代码生成器配置官方文档参考](https://baomidou.com/guides/new-code-generator/)

```java
FastAutoGenerator.create("url", "username", "password")
        .globalConfig(builder -> builder
                .author("Baomidou")
                .outputDir(Paths.get(System.getProperty("user.dir")) + "/src/main/java")
                .commentDate("yyyy-MM-dd")
        )
        .packageConfig(builder -> builder
                .parent("com.baomidou.mybatisplus")
                .entity("entity")
                .mapper("mapper")
                .service("service")
                .serviceImpl("service.impl")
                .xml("mapper.xml")
        )
        .strategyConfig(builder -> builder
                .entityBuilder()
                .enableLombok()
        )
        .templateEngine(new FreemarkerTemplateEngine())
        .execute();
```

### 4.3 Optimistic Locking

乐观锁是一种并发控制机制，用于确保在更新记录时，该记录未被其他事务修改。
MyBatis-Plus 提供了 `OptimisticLockerInnerInterceptor` 插件，使得在应用中实现乐观锁变得简单。

> **乐观锁的实现原理**  
> 乐观锁的实现通常包括以下步骤：
>
> - 读取记录时，获取当前的版本号（`version`）。
> - 在更新记录时，将这个版本号一同传递。
> - 执行更新操作时，设置 `version = newVersion` 的条件为 `version == oldVersion`。
> - 如果版本号不匹配，则更新失败。
{: .prompt-info }

#### Step 1: Add Optimistic Locker Interceptor

在 `MybatisPlusConfig` 类中添加乐观锁插件配置。

```java
@Configuration
public class MyBatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor (){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}
```

#### Step 2: Add `@Version` Annotation to Entity Class

在实体类中，使用 `@Version` 注解标记版本字段。

```java
@Data
@TableName(value = "user")
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class User {

    // ...other fields...

    @Version
    private Integer version;

}
```

Then, you can use the `updateById` method to perform an update with optimistic locking.
