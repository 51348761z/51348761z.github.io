---
title: Redis Quickstart
date: 2025-10-11 17:53:20 +0800
categories: [Redis]
tags: [redis, quickstart]
description: A quickstart guide to get you up and running with Redis.
---

Redis is an open-source, in-memory data structure store that can be used as a database, cache, and message broker. It supports various data structures such as strings, hashes, lists, sets, and more. This quickstart guide will help you get started with Redis.

## Differences between Redis and Traditional Databases(like MySQL)

1. **Data Model**: Redis is a key-value store, while traditional databases like MySQL are relational databases that use tables and rows.
2. **In-Memory Storage**: Redis primarily stores data in memory, making it extremely fast for read and write operations. MySQL, on the other hand, stores data on disk, which can be slower.
3. **Data Structures**: Redis supports various data structures such as strings, hashes, lists, sets, and sorted sets, while MySQL primarily uses tables.
4. **Scalability**: Redis is designed for high scalability and can handle large amounts of data and high throughput. MySQL can also scale, but it may require more complex configurations.
5. **Use Cases**: Redis is often used for caching, session management, real-time analytics, and message brokering. MySQL is typically used for applications that require complex queries and transactions.

Here is a table summarizing the differences:

| Feature           | Redis                        | MySQL                         |
|-------------------|------------------------------|-------------------------------|
| Data Model        | Key-Value Store              | Relational Database           |
| In-Memory Storage | Yes                          | No                            |
| Data Structures   | Strings, Hashes, Lists, Sets | Tables                        |
| Scalability       | High                         | Moderate                      |
| Use Cases         | Caching, Session Management  | Complex Queries, Transactions |

## Basic Commands

### General Commands

- `PING`: Check if the server is running.
- `ECHO <message>`: Echo the given message.
- `SELECT <db>`: Switch to a different database (0-15 by default).
- `FLUSHALL`: Remove all keys from all databases.
- `FLUSHDB`: Remove all keys from the current database.
- `INFO`: Get information and statistics about the server.
- `CONFIG GET <parameter>`: Get the value of a configuration parameter.
- `CONFIG SET <parameter> <value>`: Set the value of a configuration parameter.
- `QUIT`: Close the connection.

### Key Commands

- `SET <key> <value>`: Set the value of a key.
- `GET <key>`: Get the value of a key.
- `DEL <key>`: Delete a key.
- `EXISTS <key>`: Check if a key exists.
- `EXPIRE <key> <seconds>`: Set a timeout on a key.
- `TTL <key>`: Get the time to live for a key.
- `KEYS <pattern>`: Find all keys matching a pattern.
- `RENAME <oldkey> <newkey>`: Rename a key.
- `MGET <key1> <key2> ...`: Get the values of multiple keys.
- `MSET <key1> <value1> <key2> <value2> ...`: Set multiple keys to multiple values.
- `INCR <key>`: Increment the integer value of a key by one.
- `DECR <key>`: Decrement the integer value of a key by one.
- `APPEND <key> <value>`: Append a value to a key.
- `STRLEN <key>`: Get the length of the value stored in a key.
- `TYPE <key>`: Get the data type of the value stored in a key.
- `DUMP <key>`: Serialize the value stored in a key.
- `SETNX <key> <value>`: Set the value of a key only if it does not exist.
- `SETEX <key> <seconds> <value>`: Set the value of a key with an expiration time.
- `GETSET <key> <value>`: Set the value of a key and return its old value.

### String Commands

- `SET <key> <value>`: Set the value of a key.
- `GET <key>`: Get the value of a key.
- `APPEND <key> <value>`: Append a value to a key.
- `STRLEN <key>`: Get the length of the value stored in a key.
- `INCR <key>`: Increment the integer value of a key by one.
- `DECR <key>`: Decrement the integer value of a key by one.
- `MGET <key1> <key2> ...`: Get the values of multiple keys.
- `MSET <key1> <value1> <key2> <value2> ...`: Set multiple keys to multiple values.
- `SETNX <key> <value>`: Set the value of a key only if it does not exist.
- `SETEX <key> <seconds> <value>`: Set the value of a key with an expiration time.
- `GETSET <key> <value>`: Set the value of a key and return its old value.

### Hash Commands

- `HSET <key> <field> <value>`: Set the value of a field in a hash.
- `HGET <key> <field>`: Get the value of a field in a hash.
- `HDEL <key> <field>`: Delete a field from a hash.
- `HEXISTS <key> <field>`: Check if a field exists in a hash.
- `HGETALL <key>`: Get all fields and values in a hash.
- `HINCRBY <key> <field> <increment>`: Increment the integer value of a field in a hash by a given amount.
- `HKEYS <key>`: Get all field names in a hash.
- `HVALS <key>`: Get all values in a hash.
- `HLEN <key>`: Get the number of fields in a hash.
- `HMSET <key> <field1> <value1> <field2> <value2> ...`: Set multiple fields in a hash.
- `HMGET <key> <field1> <field2> ...`: Get the values of multiple fields in a hash.
- `HINCRBY <key> <field> <increment>`: Increment the integer value of a field in a hash by a given amount.
- `HSETNX <key> <field> <value>`: Set the value of a field in a hash only if it does not exist.

### List Commands

- `LPUSH <key> <value1> <value2> ...`: Prepend one or more values to a list.
- `RPUSH <key> <value1> <value2> ...`: Append one or more values to a list.
- `LPOP <key>`: Remove and return the first element of a list.
- `RPOP <key>`: Remove and return the last element of a list.
- `LRANGE <key> <start> <stop>`: Get a range of elements from a list.
- `LLEN <key>`: Get the length of a list.
- `LINDEX <key> <index>`: Get an element from a list by its index.
- `LSET <key> <index> <value>`: Set the value of an element in a list by its index.
- `LREM <key> <count> <value>`: Remove elements from a list.
- `BLPOP <key1> <key2> ... <timeout>`: Remove and return the first element of a list, blocking if necessary.

### Set Commands

- `SADD <key> <member1> <member2> ...`: Add one or more members to a set.
- `SREM <key> <member1> <member2> ...`: Remove one or more members from a set.
- `SPOP <key>`: Remove and return a random member from a set.
- `SMEMBERS <key>`: Get all members in a set.
- `SISMEMBER <key> <member>`: Check if a member is in a set.
- `SCARD <key>`: Get the number of members in a set.
- `SRANDMEMBER <key> [count]`: Get one or more random members from a set.
- `SUNION <key1> <key2> ...`: Get the union of multiple sets.
- `SINTER <key1> <key2> ...`: Get the intersection of multiple sets.
- `SDIFF <key1> <key2> ...`: Get the difference between multiple sets.
- `SMOVE <source> <destination> <member>`: Move a member from one set to another.

### Sorted Set Commands

- `ZADD <key> <score1> <member1> [<score2> <member2> ...]`: Add one or more members to a sorted set, or update the score of an existing member.
- `ZREM <key> <member1> <member2> ...`: Remove one or more members from a sorted set.
- `ZPOPMIN <key> [count]`: Remove and return the member with the lowest score in a sorted set.
- `ZPOPMAX <key> [count]`: Remove and return the member with the highest score in a sorted set.
- `ZRANGE <key> <start> <stop> [WITHSCORES]`: Get a range of members in a sorted set, by index.
- `ZREVRANGE <key> <start> <stop> [WITHSCORES]`: Get a range of members in a sorted set, by index, with scores ordered from high to low.
- `ZRANGEBYSCORE <key> <min> <max> [WITHSCORES] [LIMIT offset count]`: Get a range of members in a sorted set, by score.

### Pub/Sub Commands

- `PUBLISH <channel> <message>`: Post a message to a channel.
- `SUBSCRIBE <channel1> <channel2> ...`: Listen for messages published to the given channels.
- `UNSUBSCRIBE [channel1 channel2 ...]`: Stop listening for messages posted to the given channels.
- `PSUBSCRIBE <pattern1> <pattern2> ...`: Listen for messages published to channels matching the given patterns.
- `PUNSUBSCRIBE [pattern1 pattern2 ...]`: Stop listening for messages posted to channels matching the given patterns.

### RDB and AOF

RDB(Redis Database) and AOF(Append Only File) are two different persistence mechanisms in Redis.

- **RDB**: RDB creates point-in-time snapshots of your dataset at specified intervals. It is faster for loading large datasets but may lose data if Redis crashes between snapshots.
- **AOF**: AOF logs every write operation received by the server, which allows for more durable persistence. It can be configured to fsync data to disk every second, every write, or never. AOF files are generally larger than RDB files but provide better durability.

## Java Client (Jedis) Example

### Prerequisites

1. Add Jedis dependency to your `pom.xml` if you're using Maven:

    ```xml
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>4.3.1</version>
    </dependency>
    ```

2. Ensure you have a running Redis server.

3. Import necessary classes in your Java file:

    ```java
    import redis.clients.jedis.Jedis;
    ```

### Example Code

```java
public class RedisExample {
    public static void main(String[] args) {
        // Connect to Redis server
        Jedis jedis = new Jedis("localhost", 6379);
        System.out.println("Connected to Redis");
        // Set a key-value pair
        jedis.set("mykey", "Hello, Redis!");
        // Get the value of the key
        String value = jedis.get("mykey");
        System.out.println("Value of mykey: " + value);
        // Close the connection
        jedis.close();
    }
}
```

## Dive Deeper code Examples

This section provides more advanced examples of using Redis in a Java application with Spring Boot. It includes a configuration class for setting up the RedisTemplate and a utility class for common Redis operations.

Filepath: `src/main/java/com/wongs/musicgen/common/configs/RedisConfig.java`

### Redis Configuration Class

```java
package com.wongs.musicgen.common.configs;

import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;

/**
 * Redis configuration class for setting up the RedisTemplate.
 *
 * @param <V> The generic type for the value in Redis.
 */
@Configuration
public class RedisConfig<V> {
    /**
     * Creates and configures the RedisTemplate bean.
     * <p>
     * This method sets up the serializers for keys, values, hash keys, and hash values
     * to ensure proper data handling between the application and Redis.
     *
     * @param factory The Redis connection factory, automatically injected by Spring.
     * @return A configured RedisTemplate instance.
     */
    @Bean("redisTemplate")
    public RedisTemplate<String, V> redisTemplate(RedisConnectionFactory factory) {
        // Create a new RedisTemplate instance
        RedisTemplate<String, V> template = new RedisTemplate<>();
        // Set the connection factory
        template.setConnectionFactory(factory);
        // Set the serializer for keys to use strings
        template.setKeySerializer(RedisSerializer.string());
        // Set the serializer for values to use JSON
        template.setValueSerializer(RedisSerializer.json());
        // Set the serializer for hash keys to use strings
        template.setHashKeySerializer(RedisSerializer.string());
        // Set the serializer for hash values to use JSON
        template.setHashValueSerializer(RedisSerializer.json());
        // Initialize the template
        template.afterPropertiesSet();
        return template;
    }
}
```

`RedisConnectionFactory` is automatically configured by Spring Boot when you include the `spring-boot-starter-data-redis` dependency and provide the necessary Redis connection properties in your `application.properties` or `application.yml` file.  
`@Bean` annotation indicates that the `redisTemplate` method produces a bean to be managed by the Spring container.
> A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container.
{: .prompt-info}

#### How it works

1. The `RedisConfig` class is annotated with `@Configuration`, indicating that it contains bean definitions for the Spring context.
2. The `redisTemplate` method is annotated with `@Bean`, which tells Spring to create and manage a `RedisTemplate` bean.
3. The method takes a `RedisConnectionFactory` as a parameter, which is automatically provided by Spring Boot.
4. Inside the method, a new `RedisTemplate` instance is created and configured with the connection factory and appropriate serializers for keys and values.
5. Finally, the configured `RedisTemplate` instance is returned and registered as a bean in the Spring context.

### Redis Utility Class

Filepath: `src/main/java/com/wongs/musicgen/common/utils/RedisUtils.java`

```java
package com.wongs.musicgen.common.utils;

import jakarta.annotation.Resource;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.util.CollectionUtils;

import java.util.Collection;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

/**
 * A utility class for common Redis operations.
 * It is a generic class, allowing values of any type to be stored and retrieved.
 *
 * @param <V> The type of the value to be stored in Redis.
 */
public class RedisUtils<V> {
    @Resource
    private RedisTemplate<String, V> redisTemplate;

    private static final Logger LOGGER = LoggerFactory.getLogger(RedisUtils.class);

    /**
     * Deletes one or more keys from Redis.
     *
     * @param keys One or more keys to delete.
     */
    public void delete(String... keys) {
        if ((keys != null) && (keys.length > 0)) {
            if (keys.length == 1) {
                redisTemplate.delete(keys[0]);
            } else {
                redisTemplate.delete((Collection<String>) CollectionUtils.arrayToList(keys)); // Convert array to list
            }
        }
    }

    /**
     * Retrieves the value associated with a given key.
     *
     * @param key The key to retrieve.
     * @return The value associated with the key, or null if the key does not exist.
     */
    public V get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * Sets a key-value pair in Redis.
     *
     * @param key   The key to set.
     * @param value The value to associate with the key.
     * @return true if the operation was successful, false otherwise.
     */
    public boolean set(String key, V value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            LOGGER.error("Redis set operation failed for key: {}, value {}", key, value, e);
            return false;
        }
    }

    /**
     * Sets a key-value pair with an expiration time.
     *
     * @param key   The key to set.
     * @param value The value to associate with the key.
     * @param time  The expiration time in seconds. If time is less than or equal to 0, the key will not expire.
     * @return true if the operation was successful, false otherwise.
     */
    public boolean setex(String key, V value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            LOGGER.error("Redis setex operation failed for key: {}, value: {}, time: {}", key, value, time, e);
            return false;
        }
    }

    /**
     * Sets a field-value pair within a hash.
     *
     * @param key     The key of the hash.
     * @param hashKey The field within the hash.
     * @param value   The value to set for the field.
     */
    public void hset(String key, String hashKey, V value) {
        redisTemplate.opsForHash().put(key, hashKey, value);
    }

    /**
     * Retrieves the value of a field within a hash.
     *
     * @param key     The key of the hash.
     * @param hashKey The field within the hash.
     * @return The value of the field, or null if the field or key does not exist.
     */
    public V hget(String key, String hashKey) {
        return (V) redisTemplate.opsForHash().get(key, hashKey);
    }

    /**
     * Retrieves all field-value pairs from a hash.
     *
     * @param key The key of the hash.
     * @return A map containing all fields and their corresponding values from the hash.
     */
    public Map<String, V> entries(String key) {
        return (Map<String, V>) redisTemplate.opsForHash().entries(key);
    }

    /**
     * Adds an element to a sorted set with a specified score.
     *
     * @param key   The key of the sorted set.
     * @param value The value to add.
     * @param score The score to associate with the value.
     */
    public void zsetAdd(String key, V value, double score) {
        redisTemplate.opsForZSet().add(key, value, score);
    }

    /**
     * Retrieves a range of elements from a sorted set based on score.
     *
     * @param key The key of the sorted set.
     * @param min The minimum score.
     * @param max The maximum score.
     * @return A set of values within the specified score range.
     */
    public Set<V> zsetRangeByScore(String key, double min, double max) {
        return redisTemplate.opsForZSet().rangeByScore(key, min, max);
    }

    /**
     * Removes an element from a sorted set.
     *
     * @param key   The key of the sorted set.
     * @param value The value to remove.
     * @return The number of elements removed.
     */
    public Long zsetRemove(String key, V value) {
        return redisTemplate.opsForZSet().remove(key, value);
    }
}
```

> `@Resource` is used for dependency injection.  
> It injects the `RedisTemplate<String, V>` bean defined in the `RedisConfig` class into the `RedisUtils` class.
{: .prompt-info}

#### How the Utility Class Works

1. The `RedisUtils` class is a generic class that allows you to work with Redis values of any type.
2. It uses a `RedisTemplate<String, V>` to perform Redis operations, where `V` is the type of the value to be stored in Redis.
3. The class provides methods for common Redis operations, such as setting and getting keys, working with hashes, and managing sorted sets.
4. Each method includes error handling and logging to help diagnose issues during Redis operations.
