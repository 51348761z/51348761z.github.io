---
title: "Java Annotations @Repository, @Service, @Controller deep dive"
date: 2025-10-17 15:40:49 +0800
Categories: ["Java", "Annotations"]
tags: ["Java", "Annotations", "Spring", "Repository", "Service", "Controller"]
draft: true
---

在Spring框架中，构建一个结构清晰、可维护的应用程序至关重要。为了实现这一目标，Spring引入了“分层架构”的概念，并将应用程序划分为表现层（Presentation）、业务逻辑层（Business/Service）和数据访问层（Data Access/Persistence）。

`@Controller`、`@Service` 和 `@Repository` 就是Spring提供的三个核心“构造型注解”（Stereotype Annotations），专门用于标记这三个不同层中的组件。它们都是 `@Component` 注解的特化版本，除了具备将类注册为Spring Bean的基础功能外，还各自承载了特定的语义和附加功能。

### 共同点：都是 `@Component`

首先，最重要的一点是，这三个注解的定义中都包含了 `@Component` 注解。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // 核心
public @interface Service {
    // ...
}
```

这意味着，任何被 `@Controller`、`@Service` 或 `@Repository` 标记的类，都会被Spring的组件扫描（Component Scanning）机制发现，并自动在Spring容器中注册为一个Bean。因此，它们都能实现依赖注入（DI）。

### 1. `@Controller` / `@RestController`：表现层

`@Controller` 用于标记一个类作为**表现层**的组件，其主要职责是接收和处理前端发送的HTTP请求，并返回响应。

- **作用**：定义一个控制器，作为用户与应用程序交互的入口。
- **使用场景**:
  - 接收来自浏览器的HTTP请求（GET, POST, PUT, DELETE等）。
  - 调用业务逻辑层（`@Service`）的方法来处理业务。
  - 决定返回什么内容给前端，例如一个HTML页面、JSON数据或XML。
- **常用搭档**：通常与 `@RequestMapping`、`@GetMapping`、`@PostMapping` 等注解一起使用，将请求的URL映射到具体的处理方法上。

#### `@RestController`

在现代的RESTful API开发中，我们更常使用 `@RestController`。它是一个组合注解，相当于 `@Controller` + `@ResponseBody`。`@ResponseBody` 会自动将方法的返回值（如一个Java对象）序列化为JSON或XML格式的响应体，极大地简化了API的开发。

**示例代码：**

```java
import org.springframework.web.bind.annotation.*;

@RestController // 标记为RESTful控制器
@RequestMapping("/api/users") // 所有请求都以 /api/users 为前缀
public class UserController {

    private final UserService userService;

    // 通过构造函数注入UserService
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public UserDTO getUserById(@PathVariable Long id) {
        // 调用Service层获取业务数据
        return userService.findUserById(id);
    }

    @PostMapping
    public UserDTO createUser(@RequestBody CreateUserRequest request) {
        // 调用Service层创建新用户
        return userService.createUser(request);
    }
}
```

### 2. `@Service`：业务逻辑层

`@Service` 用于标记一个类作为**业务逻辑层**的组件。它封装了应用程序的核心业务规则和流程。

- **作用**：定义一个服务，用于处理具体的业务逻辑。它是表现层和数据访问层之间的桥梁。
- **使用场景**:
  - 编排和组合多个数据访问操作。
  - 实现复杂的业务计算和校验。
  - 管理事务（通常通过 `@Transactional` 注解）。
  - 将控制器传递来的请求数据（DTO）转换为持久化对象（Entity）。

**示例代码：**

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service // 标记为业务逻辑组件
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;

    // 注入UserRepository
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDTO findUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new UserNotFoundException("User not found with id: " + id));
        // 将Entity转换为DTO
        return convertToDTO(user);
    }

    @Override
    @Transactional // 声明这是一个事务性操作
    public UserDTO createUser(CreateUserRequest request) {
        // 业务逻辑：例如检查用户名是否已存在
        if (userRepository.existsByUsername(request.getUsername())) {
            throw new UsernameAlreadyExistsException("Username is already taken.");
        }
        
        User newUser = new User();
        newUser.setUsername(request.getUsername());
        newUser.setPassword(encodePassword(request.getPassword())); // 加密密码
        
        User savedUser = userRepository.save(newUser);
        
        return convertToDTO(savedUser);
    }
    
    // ... 其他辅助方法
}
```

### 3. `@Repository`：数据访问层

`@Repository` 用于标记一个类作为**数据访问层**的组件（也称为DAO - Data Access Object）。它专门负责与数据库进行交互。

- **作用**：定义一个数据仓库，封装所有与数据持久化相关的操作（CRUD -增删改查）。
- **使用场景**:
  - 直接与数据库、缓存或其他数据存储进行交互。
  - 在Spring Data JPA中，通常标记在继承 `JpaRepository` 的接口上（虽然Spring Data JPA会自动为这些接口创建代理Bean，但标记上可以增强代码清晰度）。
- **特殊功能：异常转换（Exception Translation）**
    这是 `@Repository` 相比 `@Component` 最重要的一个附加功能。Spring会为被 `@Repository` 注解的Bean添加一个AOP切面，自动将特定于数据访问技术（如JDBC、Hibernate、JPA）的已检查异常（Checked Exceptions，如 `SQLException`）转换为Spring统一的、非检查的 `DataAccessException` 异常体系中的某个子类。这使得业务逻辑层可以从底层数据访问技术的异常中解耦。

**示例代码：**

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository // 标记为数据访问组件
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Spring Data JPA会根据方法名自动生成SQL查询
    boolean existsByUsername(String username);
    
    Optional<User> findByUsername(String username);
}
```

如果需要自定义实现，可以这样写：

```java
@Repository
public class CustomUserRepositoryImpl implements CustomUserRepository {
    
    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public void someCustomMethod() {
        // 在这里，如果发生SQLException，Spring会将其转换为DataAccessException
        entityManager.createNativeQuery("...").executeUpdate();
    }
}
```

### 总结对比

| 注解 | 所属层 | 主要职责 | 附加功能 |
| :--- | :--- | :--- | :--- |
| **`@Controller`** | 表现层 | 接收HTTP请求，返回响应 | - |
| **`@RestController`** | 表现层 | `@Controller` + `@ResponseBody`，专用于REST API | 自动序列化返回值为JSON/XML |
| **`@Service`** | 业务逻辑层 | 封装核心业务逻辑，编排数据操作 | 语义化，表明这是业务核心 |
| **`@Repository`** | 数据访问层 | 与数据库交互，实现数据持久化 | **异常转换**，将底层异常转为Spring的`DataAccessException` |

通过正确使用这三个注解，你可以构建一个职责分明、易于测试和维护的经典三层架构Spring应用。
