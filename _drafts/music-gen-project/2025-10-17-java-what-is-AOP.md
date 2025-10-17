---
title: "Java AOP(Aspect Oriented Programming) learnings"
date: 2025-10-17 16:59:45 +0800
categories: ["Java", "AOP"]
tags: ["Java", "AOP", "Aspect Oriented Programming", "Annotation", "Meta-Annotation"]
draft: true
description: "Understanding the basics of AOP in Java, including key concepts, types of advice, and practical examples using Spring AOP."
---

## 什么是AOP？

**AOP（Aspect-Oriented Programming）**，即**面向切面编程**，是一种编程思想，旨在通过分离**横切关注点（Cross-Cutting Concerns）**来提高程序的模块化程度。

在传统的**OOP（Object-Oriented Programming，面向对象编程）**中，我们将程序分解为一个个的对象，每个对象封装了自身的属性和行为。这在处理核心业务逻辑时非常有效。然而，在实际应用中，存在一些功能需要“横跨”多个对象和模块，例如：

* **日志记录**：在很多方法的开始和结束时需要记录日志。
* **权限校验**：在执行敏感操作前需要检查用户权限。
* **事务管理**：在数据库操作前后需要开启和提交/回滚事务。
* **性能监控**：需要计算某些方法的执行时间。

这些功能被称为**横切关注点**，因为它们像切片一样“切入”到我们多个核心业务逻辑中。如果我们在每个需要它们的方法里都手动编写这些代码，会导致：

1. **代码重复**：大量样板代码散布在各处。
2. **核心业务逻辑混杂**：业务方法的代码不纯粹，难以阅读和维护。
3. **修改困难**：如果日志格式或权限逻辑需要变更，必须修改所有相关方法。

**AOP的目标就是解决这个问题**。它允许我们将这些横切关注点从业务逻辑中抽离出来，封装成一个独立的模块（称为“切面”），然后以声明的方式“织入”到需要它的地方，而无需修改业务代码本身。

## AOP核心术语

要理解AOP，必须先掌握以下几个关键概念：

1. **切面（Aspect）**:
    * 对横切关注点的模块化封装。一个切面可以包含多个通知和切点。在Spring中，就是一个带有 `@Aspect` 注解的Java类。

2. **连接点（Join Point）**:
    * 程序执行过程中的一个特定点，例如方法的调用、异常的抛出等。在Spring AOP中，**连接点总是指方法的执行**。它是切面可以“插入”代码的地方。

3. **通知（Advice）**:
    * 切面在特定连接点上执行的**具体操作**。也就是我们抽离出来的日志、安全、事务等逻辑代码。Spring提供了5种类型的通知。

4. **切点（Pointcut）**:
    * 一个**匹配连接点的表达式**。它定义了通知应该被应用到哪些连接点（即哪些方法）上。我们可以把它看作一个“查询”，用于筛选出我们感兴趣的方法。

5. **目标对象（Target Object）**:
    * 被一个或多个切面所“通知”的对象。也就是包含我们核心业务逻辑的那个原始对象。

6. **代理（Proxy）**:
    * AOP框架创建的一个对象，用于包装目标对象。代理对象在外部看起来和目标对象一样，但它内部包含了切面的逻辑。当调用代理对象的方法时，它会在执行目标对象方法的前后，执行切面的通知。Spring AOP默认使用**JDK动态代理**或**CGLIB代理**。

7. **织入（Weaving）**:
    * 将切面应用到目标对象上，并创建出代理对象的过程。织入可以在编译时、加载时或运行时完成。**Spring AOP是在运行时进行织入的**。

## AOP通知的类型

Spring AOP通过注解定义了5种通知类型：

* `@Before`：**前置通知**。在目标方法执行**之前**运行。
* `@After`：**后置通知**。在目标方法执行**之后**运行，无论方法是正常返回还是抛出异常。
* `@AfterReturning`：**返回后通知**。仅在目标方法**成功执行并返回**后运行。可以获取方法的返回值。
* `@AfterThrowing`：**异常后通知**。仅在目标方法**抛出异常**后运行。可以获取抛出的异常。
* `@Around`：**环绕通知**。最强大的通知类型。它“包围”了整个目标方法的执行。你可以在方法执行前后自定义操作，甚至可以决定是否执行目标方法、修改返回值或抛出异常。

## AOP的典型使用场景

* **统一日志记录**：为指定的方法调用自动记录出入参和执行时间。
* **声明式事务管理**：通过 `@Transactional` 注解轻松管理数据库事务。
* **统一权限校验**：在执行控制器方法前检查用户是否登录、是否有权访问。
* **缓存控制**：在查询方法执行前检查缓存，如果命中则直接返回，否则执行方法并将结果存入缓存。
* **性能监控**：统计关键业务方法的调用次数和平均耗时。
* **统一异常处理**：将底层的特定异常转换为统一的业务异常。

## 示例代码：实现一个方法执行时间日志切面

下面是一个非常经典的例子，演示如何使用AOP来记录方法的执行时间。

**1. 添加依赖**

确保你的 `pom.xml` 中有 `spring-boot-starter-aop` 依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**2. 定义一个Service**

我们有一个简单的服务，其中一个方法我们希望监控其执行时间。

```java
@Service
public class MyBusinessService {

    public String doSomething(String param) {
        // 模拟耗时操作
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("核心业务逻辑执行: " + param);
        return "Success";
    }

    public void anotherMethod() {
        System.out.println("这个方法不被监控");
    }
}
```

**3. 创建切面（Aspect）**

这是AOP的核心。我们创建一个类，用 `@Aspect` 和 `@Component` 标记它，并定义切点和通知。

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Aspect      // 声明这是一个切面类
@Component   // 将其注册为Spring Bean
public class PerformanceLogAspect {

    /**
     * 定义一个切点表达式。
     * execution(* com.example.demo.MyBusinessService.doSomething(..))
     * - execution(): 表示这是一个方法执行的连接点。
     * - *: 匹配任意返回值类型。
     * - com.example.demo.MyBusinessService.doSomething: 指定要匹配的完整方法名。
     * - (..): 匹配任意数量、任意类型的参数。
     */
    @Pointcut("execution(* com.example.demo.MyBusinessService.doSomething(..))")
    public void myServiceMethods() {}

    /**
     * 定义环绕通知，并关联到上面的切点。
     * ProceedingJoinPoint 参数是必需的，它代表了被通知的目标方法。
     */
    @Around("myServiceMethods()")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();

        // 调用 joinPoint.proceed() 来执行原始的目标方法
        Object result = joinPoint.proceed();

        long endTime = System.currentTimeMillis();
        long executionTime = endTime - startTime;

        System.out.println(
            "AOP日志: " + joinPoint.getSignature() + " 执行耗时 " + executionTime + "ms"
        );

        // 必须返回原始方法的返回值
        return result;
    }
}
```

**4. 运行和结果**

当你调用 `myBusinessService.doSomething("test")` 时，控制台会输出：

```shell
核心业务逻辑执行: test
AOP日志: String com.example.demo.MyBusinessService.doSomething(String) 执行耗时 1005ms
```

而调用 `myBusinessService.anotherMethod()` 时，则不会触发AOP日志，因为它的方法签名没有被切点表达式匹配到。

这个例子完美地展示了AOP的强大之处：我们**没有修改一行 `MyBusinessService` 的代码**，就为其添加了性能监控的功能。

---

### 结合AOP：使用自定义注解驱动切面

在之前的AOP示例中，我们使用了一个切点表达式 `execution(* com.example.demo.MyBusinessService.doSomething(..))` 来硬编码指定要拦截的方法。这种方式虽然有效，但存在两个问题：

1. **耦合性高**：切面代码与具体的业务方法路径紧密耦合。如果方法名或路径改变，切点也必须修改。
2. **意图不明确**：当其他人阅读业务代码时，无法直接看出这个方法被AOP增强了。

一个更优雅的解决方案是：**创建一个自定义注解，用它来标记我们希望应用切面的方法，然后让切点直接匹配这个注解。**

这样，注解就成了一个“开关”或“标记”，业务代码通过“声明”自己需要某种横切功能，而AOP切面则负责响应这个声明。

## 元注解：创建注解的基石

在创建自己的注解之前，我们需要先了解**元注解（Meta-Annotations）**。它们是Java提供的、用于“修饰”其他注解的特殊注解，用来定义我们自定义注解的行为。

最重要的元注解有以下四个：

1. **`@Retention`**: 定义注解的**生命周期**。它有三个可选值：
    * `RetentionPolicy.SOURCE`: 注解仅存在于源代码中，编译后被丢弃。主要供编译器或Lombok这类工具使用（如 `@SuppressWarnings`）。
    * `RetentionPolicy.CLASS`: 注解被保留在 `.class` 文件中，但JVM在运行时无法获取它。这是默认值，很少直接使用。
    * `RetentionPolicy.RUNTIME`: 注解被保留在 `.class` 文件中，并且在运行时可以通过**反射（Reflection）**获取。**这是AOP和大多数框架使用的策略，因为它们需要在程序运行时读取注解信息。**

2. **`@Target`**: 定义注解可以被应用在**哪些程序元素上**。可以是一个或多个值：
    * `ElementType.METHOD`: 只能用于方法（AOP中最常用）。
    * `ElementType.TYPE`: 可用于类、接口、枚举。
    * `ElementType.FIELD`: 可用于字段（成员变量）。
    * `ElementType.PARAMETER`: 可用于方法参数。
    * `ElementType.CONSTRUCTOR`: 可用于构造函数。
    * *...等等。*

3. **`@Documented`**:
    * 一个标记注解。表示在使用 Javadoc 工具生成文档时，应该将此注解信息也包含在内，增强文档的可读性。

4. **`@Inherited`**:
    * 一个标记注解。如果一个类使用了被 `@Inherited` 修饰的注解，那么其所有子类将自动继承这个注解。

## 示例：创建一个 `@LogExecutionTime` 注解来驱动AOP

让我们来改造之前的性能监控示例。

### 第一步：创建自定义注解

我们定义一个 `@LogExecutionTime` 注解。它的作用就是标记一个方法需要被我们的性能监控切面所拦截。

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 自定义注解，用于标记需要记录执行时间的方法。
 */
@Retention(RetentionPolicy.RUNTIME) // 必须是 RUNTIME，这样AOP才能在运行时看到它
@Target(ElementType.METHOD)         // 这个注解只能用在方法上
public @interface LogExecutionTime {
}
```

这个注解非常简单，它只是一个纯粹的“标记”，没有任何元素。

### 第二步：在业务代码中使用注解

现在，我们不再关心方法名叫什么，只需要给希望监控的方法贴上 `@LogExecutionTime` 标签即可。

```java
@Service
public class MyBusinessService {

    @LogExecutionTime // 标记这个方法需要记录执行时间
    public String doSomething(String param) {
        // 模拟耗时操作
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("核心业务逻辑执行: " + param);
        return "Success";
    }

    @LogExecutionTime // 我们现在也可以轻松地监控这个方法了
    public void anotherMethod() {
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("另一个被监控的方法");
    }
}
```

### 第三步：修改切面，使用注解作为切点

最后，我们修改 `PerformanceLogAspect`，将切点表达式从 `execution(...)` 改为 `@annotation(...)`。

`@annotation()` 表达式用于匹配所有被指定注解标记的连接点（方法）。

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class PerformanceLogAspect {

    /**
     * 定义一个新的切点，匹配所有被 @LogExecutionTime 注解标记的方法。
     * 注意路径要写全。
     */
    @Pointcut("@annotation(com.example.demo.LogExecutionTime)")
    public void logExecutionTimeMethods() {}

    /**
     * 环绕通知现在关联到新的切点上。
     */
    @Around("logExecutionTimeMethods()")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();

        Object result = joinPoint.proceed(); // 执行目标方法

        long endTime = System.currentTimeMillis();
        long executionTime = endTime - startTime;

        System.out.println(
            "AOP日志 (通过注解): " + joinPoint.getSignature() + " 执行耗时 " + executionTime + "ms"
        );

        return result;
    }
}
```

### 运行结果与优势分析

现在，当你调用 `doSomething()` 和 `anotherMethod()` 时，它们都会被AOP拦截并打印执行时间。

**这种模式的巨大优势在于：**

1. **高度解耦**：切面不再关心业务逻辑的类名或方法名，只关心“是否需要记录执行时间”这个**意图**。
2. **代码即文档**：当你在一个方法上看到 `@LogExecutionTime`，你立刻就能明白它的一个横切关注点是性能监控，代码的可读性大大增强。
3. **极易扩展**：未来如果有一个全新的服务 `NewService` 中的某个方法也需要性能监控，你唯一要做的就是在那个方法上添加 `@LogExecutionTime` 注解，无需改动任何AOP代码。
4. **关注点分离**：业务开发者可以专注于业务逻辑，只需在需要时“声明”需要某个AOP功能即可；而AOP的开发者则专注于横切功能的实现，双方互不干扰。

这正是AOP与自定义注解相结合的魅力所在，也是Spring等框架实现其强大功能的核心思想之一。
