---
title: 'Music-Gen: User Info Module'
date: 2025-10-13 12:45:26 +0800
categories: [music-gen, user-module]
tags: [music-gen, user-module, token, java]
---

TODO: Write the content for the Music-Gen User Info Module.

## 用户身份验证：从 Session 到 Token

在 Web 开发中，一个核心问题是如何在多次请求之间识别用户身份。这背后的挑战源于 HTTP 协议本身的一个基本特性：**无状态（Stateless）**。

### 1. 核心问题：HTTP 的无状态性

“无状态”意味着服务器不会保存客户端在多次请求之间的任何状态或上下文信息。每一次从客户端发来的请求，对于服务器来说都是一个全新的、独立的事务。服务器处理完请求并发送响应后，就会完全“忘记”这个客户端。

这就带来一个问题：用户登录成功后，当他再次请求其他页面或数据时，服务器如何知道他还是刚才那个已经登录的用户呢？为了解决这个问题，诞生了两种主流的技术方案：Session 和 Token。

### 2. 传统方案：有状态的 Session

`Session`（会话）是服务器用来跟踪和维护特定用户状态的一种传统机制，它在服务器端创建了一个“有状态”的上下文。

#### Session 如何工作？

1. **创建会话**：用户成功登录后，服务器为其创建一个 Session（一个存储在服务器上的数据结构），用来存放用户信息（如用户ID、登录状态等）。
2. **生成并发送 Session ID**：服务器为该 Session 生成一个唯一的 Session ID，并通过 Cookie 发送给客户端浏览器。
3. **客户端携带 Session ID**：浏览器在后续的每次请求中，都会自动携带包含 Session ID 的 Cookie。
4. **服务器验证**：服务器根据收到的 Session ID，在自身存储中找到对应的 Session 数据，从而识别用户身份。

简单来说，**Session 就像是游乐园的腕带**：你在入口处买票（登录），工作人员给你戴上一个腕带（Session ID）。之后你玩任何项目，工作人员只需检查你的腕带，就能从电脑系统里查到你的购票信息。

#### Session 的主要缺点

Session 的主要问题在于**扩展性差**。因为 Session 数据通常存储在单台服务器的内存中。在需要多台服务器进行负载均衡的现代应用架构中，请求可能被分发到不同的服务器上，这会导致一台服务器无法识别另一台服务器创建的 Session，除非引入额外的复杂技术（如 Session 复制或共享存储）。

### 3. 现代方案：无状态的 Token

为了解决 Session 的扩展性问题，`Token`（令牌）作为一种无状态的身份验证方案应运而生。

#### Token 是什么？

`Token` 是一个加密的字符串，它本身就包含了足够的用户身份信息。服务器通过验证这个字符串的合法性来确认用户身份，而无需在自己这边存储任何会话数据。一个非常流行的 Token 标准是 **JWT (JSON Web Token)**。

#### JWT (JSON Web Token) 示例

一个 JWT 看起来像这样的一长串字符，由两个点（`.`）分隔成三个部分：
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiJ1c2VyLTEyMyIsInVzZXJuYW1lIjoiQWxpY2UifQ.q_tCgR_p-QoZ_wY-V2a_bC4n_xRz_bE5a_cE6wF_dE`

这三部分分别是：

1. **Header (头部)**: 描述 JWT 元数据，如签名算法。
2. **Payload (载荷)**: 包含实际的用户信息和声明（Claims）。
3. **Signature (签名)**: 用于验证 Token 未被篡改的签名。

```java
// 伪代码示例 (使用 Java 的 jjwt 库)
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.security.Keys;
import javax.crypto.SecretKey;
import java.util.Date;

// 1. 定义用于签名的密钥 (必须保存在服务器上，且不能泄露)
// 使用 Keys.secretKeyFor 安全地生成一个密钥
// SecretKey secretKey = Keys.secretKeyFor(SignatureAlgorithm.HS256); // jjwt 0.12 lower
SecretKey secretKey = Jwts.SIG.HS256.key().build(); // jjwt 0.12+



// 2. 定义要包含在 Token 中的用户信息 (Payload/Claims)
String userId = "user-123";
String username = "Alice";

// 3. 生成 Token (设置有效期为 1 小时)
long nowMillis = System.currentTimeMillis();
Date now = new Date(nowMillis);
long expMillis = nowMillis + 3600000; // 1小时后过期
Date exp = new Date(expMillis);

String token = Jwts.builder()
    .claim("username", username) // 添加自定义信息
    .subject(userId) // 'subject' 通常用于存放用户唯一标识
    .issuedAt(now) // 设置签发时间
    .expiration(exp) // 设置过期时间
    .signWith(secretKey) // 使用密钥签名
    .compact();

System.out.println("生成的 Token: " + token);

// --- 稍后，当客户端带着 Token 发来请求时 ---

// 4. 服务器从请求头中获取 Token 并进行验证
try {
    // Jwts.parser() 会自动使用密钥验证签名和有效期
    Claims claims = Jwts.parser()
        .verifyWith(secretKey)
        .build()
        .parseSignedClaims(token)
        .getPayload();

    System.out.println("验证成功，用户ID: " + claims.getSubject());
    System.out.println("验证成功，用户名: " + claims.get("username", String.class));
    // 验证成功，可以根据 claims.getSubject() 去处理后续业务逻辑

} catch (Exception e) {
    System.err.println("Token 验证失败: " + e.getMessage());
    // 如果验证失败（如签名错误、Token 过期），则拒绝请求
}
```

#### Token 的工作流程

1. **用户登录**：用户使用凭据（如用户名和密码）登录。
2. **服务器签发 Token**：服务器验证凭据成功后，生成一个 Token（其中包含了用户ID、权限、过期时间等信息），并将其返回给客户端。
3. **客户端存储 Token**：客户端（如浏览器或手机App）将 Token 存储起来（常见于 `localStorage` 或请求头中）。
4. **携带 Token 发起请求**：在访问受保护的资源时，客户端在请求头（通常是 `Authorization` 头）中附带上这个 Token。
5. **服务器验证 Token**：服务器收到请求后，验证 Token 的签名是否有效、是否过期。验证通过后，即可信任 Token 中的信息并处理请求。

**Token 就像是一张详细的门票**：门票上直接印着你的姓名、允许游玩的项目和有效期。任何一个检票员（服务器）拿到这张票，只需验证票的真伪，就能知道所有信息，而无需去查中央电脑系统。

### 4. 深入辨析：关于 Token 的几个关键问题

#### “无状态”是否意味着服务器什么都不存？

不完全是。更准确的说法是：**服务器不需要为每个用户保存会话状态（Session State），但它必须保存用于验证 Token 所需的全局信息。**

* **必须保存**：用于给 Token 签名的**密钥（Secret Key）**。服务器在签发 Token 时用它加密，在验证 Token 时也用它解密和校验。这个密钥是全局共享的，而不是为每个用户单独创建的。
* **无需保存**：每个用户的登录状态、购物车内容等会话数据。这些信息要么包含在 Token 里，要么由客户端自己管理。

#### Token 一旦签发，如何让它失效？

纯粹的无状态模型有一个缺点：Token 在过期前始终有效。如果用户修改密码或被封禁，我们希望他当前的 Token 立刻失效。

为了解决这个问题，可以引入**“黑名单（Blacklist）”机制**。服务器可以维护一个列表，记录所有被强制吊销的 Token。在验证时，除了检查签名和有效期，还需检查该 Token 是否在黑名单上。这虽然给无状态模型增加了一点“状态”，但相比管理所有用户的 Session，这种开销要小得多。

---

## 用户密码加密处理

在用户注册或修改密码时，直接存储明文密码是非常危险的。为了保护用户的密码安全，通常会对密码进行加密处理。常见的做法是使用哈希函数（如 bcrypt、scrypt 或 Argon2）对密码进行单向加密，并添加“盐”（salt）来防止彩虹表攻击。
> **什么是彩虹表攻击？**  
> 彩虹表攻击是一种通过预计算大量可能的密码及其对应哈希值来破解密码的方法。攻击者可以使用这些预计算的哈希值来快速查找用户的密码，而不需要逐个尝试所有可能的密码组合。  
> **盐（salt）**: 是在密码哈希前添加的一段随机数据，即使两个用户使用相同的密码，添加不同的盐后生成的哈希值也会不同，从而增强了密码的安全性。
{: .prompt-info}

### 一个彩虹表攻击的简单例子

假设有一个网站，它使用了一种**没有加盐**的简单哈希算法（例如古老的 MD5）来存储密码。

#### 场景一：没有加盐

1. **用户注册**：
    * 用户 **Alice** 的密码是 `123456`。数据库存储的哈希值是 `hash("123456")` -> `e10adc3949ba59abbe56e057f20f883e`。
    * 用户 **Bob** 的密码也是 `123456`。数据库存储的哈希值也是 `hash("123456")` -> `e10adc3949ba59abbe56e057f20f883e`。

2. **攻击者行动**：
    * 攻击者通过漏洞窃取了网站的数据库，得到了所有用户的密码哈希列表。
    * 攻击者**事先已经准备好了一张“彩虹表”**。这张表就像一个巨大的字典，里面存着亿万个常用密码和它们对应的哈希值。
    * 攻击者拿着从数据库偷来的哈希值 `e10adc3949ba59abbe56e057f20f883e`，直接去查他的彩虹表，瞬间就查到这个哈希值对应的明文密码是 `123456`。

3. **结果**：攻击者成功破解了 Alice 和 Bob 的密码，整个过程几乎没有进行任何计算，只是做了一次快速的查找。

---

#### 场景二：加盐之后

现在，网站升级了安全策略，使用了**加盐**的哈希算法（如 BCrypt）。

1. **用户注册**：
    * 用户 **Alice** 的密码是 `123456`。系统为她生成一个随机盐 `salt_A`，数据库存储 `hash("123456" + "salt_A")` -> `hash_A`。
    * 用户 **Bob** 的密码也是 `123456`。系统为他生成另一个随机盐 `salt_B`，数据库存储 `hash("123456" + "salt_B")` -> `hash_B`。
    * **关键点**：尽管 Alice 和 Bob 的密码相同，但因为盐不同，他们最终存储在数据库中的哈希值 `hash_A` 和 `hash_B` 是完全不同的。

2. **攻击者行动**：
    * 攻击者再次偷走了数据库，得到了 `hash_A` 和 `hash_B`。
    * 他尝试去查他的彩虹表，但完全找不到匹配项。因为他的彩虹表是基于 `hash(密码)` 计算的，而数据库里存的是 `hash(密码 + 盐)`。

3. **结果**：彩虹表完全失效了。攻击者如果想破解密码，就必须针对**每一个用户不同的盐**去重新计算一张新的彩虹表，这在计算上是不可行的。

**总结：盐（Salt）的作用就是让每个用户的密码哈希都变得独一无二，从而让攻击者预先计算好的彩虹表彻底失效。**

---

### 在 Spring Boot 中实现密码加密

在 Spring Boot 项目中，处理用户密码加密的标准方式是使用 Spring Security 框架提供的 `PasswordEncoder`，最推荐的实现是 `BCryptPasswordEncoder`。

#### 1. 配置 PasswordEncoder Bean

首先，在你的一个配置类（例如 `SecurityConfig.java`）中，将 `BCryptPasswordEncoder` 注册为一个 Bean，以便在整个应用中注入和使用。

```java
// 建议放在项目的安全配置类中，如 SecurityConfig.java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    // ... 其他 Spring Security 配置
}
```

#### 2. 在业务代码中使用

然后，在需要处理密码的地方（通常是用户服务类 `UserService`），注入这个 `PasswordEncoder` Bean。

* **注册时**：使用 `encode()` 方法将明文密码加密。
* **登录时**：使用 `matches()` 方法比对用户输入的明文密码和数据库中存储的加密密码。

```java
// 建议放在用户相关的服务类中，如 UserService.java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private final PasswordEncoder passwordEncoder;

    @Autowired
    public UserService(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }

    /**
     * 注册新用户时，对密码进行加密
     * @param rawPassword 用户输入的明文密码
     * @return 加密后的密码字符串，用于存入数据库
     */
    public String registerUser(String username, String rawPassword) {
        String encodedPassword = passwordEncoder.encode(rawPassword);
        // 接下来，你应该将 username 和 encodedPassword 保存到数据库的用户表中
        System.out.println("为用户 " + username + " 加密密码: " + encodedPassword);
        return encodedPassword;
    }

    /**
     * 用户登录时，验证密码是否正确
     * @param rawPassword 用户输入的明文密码
     * @param encodedPasswordFromDb 从数据库中取出的已加密密码
     * @return 如果匹配则返回 true，否则返回 false
     */
    public boolean loginUser(String rawPassword, String encodedPasswordFromDb) {
        boolean isMatch = passwordEncoder.matches(rawPassword, encodedPasswordFromDb);
        System.out.println("密码验证结果: " + isMatch);
        return isMatch;
    }
}
```

> **重要提示**：`BCryptPasswordEncoder` 已经内置了加盐处理，你无需手动管理盐。  
> `encode()` 方法每次调用都会生成一个不同的哈希值，但 `matches()` 方法依然可以正确地进行比对。
{: .prompt-tip}

---
