---
title: "Claude Code 在 Spring Boot 项目中的深度实战 06：基于 Testcontainers 生成集成测试套件"
date: 2026-06-20
categories:
  - Java
  - Spring Boot
  - Claude Code
tags:
  - Testcontainers
  - 集成测试
  - MySQL
  - RestAssured
---

## 一、案例背景

本案例是综合级实战，预计耗时 30 分钟。目标是为 `UserController` 生成覆盖全部接口的集成测试套件。测试环境使用 Spring Boot 3.x、Java 17、JUnit 5、Testcontainers 和 MySQL 8.0。请求发送工具可以选择 RestAssured 或 `TestRestTemplate`，本文使用 RestAssured 演示。

与普通单元测试不同，集成测试会启动 Spring 容器，并通过真实 HTTP 请求调用 Controller。使用 Testcontainers 可以在测试时动态启动 MySQL 容器，避免依赖开发者本机数据库，使测试更稳定、更接近 CI 环境。

---

## 二、Maven 依赖示例

`pom.xml` 中需要包含：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>mysql</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

如果使用 Gradle：

```bash
gradle test --tests UserControllerIntegrationTest
```

Maven 单测命令：

```bash
mvn -Dtest=UserControllerIntegrationTest test
```

---

## 三、完整提示词

在 Claude Code 中输入：

```text
请为 UserController 生成集成测试类 UserControllerIntegrationTest。

项目环境：
- Spring Boot 3.x
- Java 17
- JUnit 5
- Spring Data JPA
- MySQL 8.0
- Testcontainers
- RestAssured

请只读取以下文件：
- src/main/java/com/yourcompany/demo/controller/UserController.java
- src/main/java/com/yourcompany/demo/entity/User.java
- src/main/java/com/yourcompany/demo/common/Result.java
- src/main/java/com/yourcompany/demo/repository/UserRepository.java
- pom.xml

测试要求：
1. 使用 @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)。
2. 使用 Testcontainers 启动 mysql:8.0 容器。
3. 使用 @DynamicPropertySource 注入 spring.datasource.url、username、password。
4. 使用 RestAssured 发送 HTTP 请求。
5. 覆盖 save、findById、findAll、update、deleteById 五个接口。
6. 断言 Result<T> 中的 code、msg、data 字段。
7. 每个测试前清理 UserRepository 数据。
8. 不要依赖本机 MySQL。
9. 生成后给出运行命令。
```

这个提示词把测试边界、依赖、读取范围、断言字段和运行命令全部明确了。Claude Code 生成的测试更容易一次可用。

---

## 四、生成的测试类代码

```java
package com.yourcompany.demo.controller;

import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;
import static org.hamcrest.Matchers.notNullValue;

import com.yourcompany.demo.entity.User;
import com.yourcompany.demo.repository.UserRepository;
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import java.util.HashMap;
import java.util.Map;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.MySQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserControllerIntegrationTest {

    @Container
    static final MySQLContainer<?> MYSQL_CONTAINER = new MySQLContainer<>("mysql:8.0")
            .withDatabaseName("demo_test")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void registerProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", MYSQL_CONTAINER::getJdbcUrl);
        registry.add("spring.datasource.username", MYSQL_CONTAINER::getUsername);
        registry.add("spring.datasource.password", MYSQL_CONTAINER::getPassword);
        registry.add("spring.jpa.hibernate.ddl-auto", () -> "create-drop");
    }

    @LocalServerPort
    private int port;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        RestAssured.port = port;
        userRepository.deleteAll();
    }

    @Test
    void shouldSaveUser() {
        Map<String, Object> request = new HashMap<>();
        request.put("username", "zhangsan");
        request.put("email", "zhangsan@example.com");
        request.put("enabled", true);

        given()
                .contentType(ContentType.JSON)
                .body(request)
        .when()
                .post("/api/users")
        .then()
                .statusCode(200)
                .body("code", equalTo(200))
                .body("msg", equalTo("success"))
                .body("data.id", notNullValue())
                .body("data.username", equalTo("zhangsan"))
                .body("data.email", equalTo("zhangsan@example.com"))
                .body("data.enabled", equalTo(true));
    }

    @Test
    void shouldFindUserById() {
        User user = userRepository.save(User.builder()
                .username("lisi")
                .email("lisi@example.com")
                .enabled(true)
                .build());

        given()
        .when()
                .get("/api/users/{id}", user.getId())
        .then()
                .statusCode(200)
                .body("code", equalTo(200))
                .body("data.id", equalTo(user.getId().intValue()))
                .body("data.username", equalTo("lisi"));
    }

    @Test
    void shouldFindAllUsers() {
        userRepository.save(User.builder()
                .username("user1")
                .email("user1@example.com")
                .enabled(true)
                .build());
        userRepository.save(User.builder()
                .username("user2")
                .email("user2@example.com")
                .enabled(false)
                .build());

        given()
        .when()
                .get("/api/users")
        .then()
                .statusCode(200)
                .body("code", equalTo(200))
                .body("data.size()", equalTo(2));
    }

    @Test
    void shouldUpdateUser() {
        User user = userRepository.save(User.builder()
                .username("oldName")
                .email("old@example.com")
                .enabled(true)
                .build());

        Map<String, Object> request = new HashMap<>();
        request.put("username", "newName");
        request.put("email", "new@example.com");
        request.put("enabled", false);

        given()
                .contentType(ContentType.JSON)
                .body(request)
        .when()
                .put("/api/users/{id}", user.getId())
        .then()
                .statusCode(200)
                .body("code", equalTo(200))
                .body("data.username", equalTo("newName"))
                .body("data.email", equalTo("new@example.com"))
                .body("data.enabled", equalTo(false));
    }

    @Test
    void shouldDeleteUserById() {
        User user = userRepository.save(User.builder()
                .username("deleteMe")
                .email("delete@example.com")
                .enabled(true)
                .build());

        given()
        .when()
                .delete("/api/users/{id}", user.getId())
        .then()
                .statusCode(200)
                .body("code", equalTo(200))
                .body("msg", equalTo("success"));

        given()
        .when()
                .get("/api/users")
        .then()
                .statusCode(200)
                .body("data.size()", equalTo(0));
    }
}
```

---

## 五、常见生成问题

### 1. 容器没有启动

根因通常是 Docker Desktop 未启动。解决：先启动 Docker，再执行：

```bash
mvn -Dtest=UserControllerIntegrationTest test
```

### 2. 表不存在

根因是测试环境没有正确设置：

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: create-drop
```

也可以通过 `@DynamicPropertySource` 注入。

### 3. Long 类型断言失败

RestAssured 读取 JSON 数字时可能把 Long 转成 Integer。示例中使用：

```java
.body("data.id", equalTo(user.getId().intValue()))
```

如果 ID 超过 Integer 范围，应改用更稳妥的提取后断言。

---

## 六、运行命令

Maven：

```bash
mvn -Dtest=UserControllerIntegrationTest test
```

Gradle：

```bash
gradle test --tests UserControllerIntegrationTest
```

如果测试失败，把完整日志贴给 Claude Code：

```text
下面是 UserControllerIntegrationTest 的失败日志。请只修复测试或 UserController 相关问题，不要修改其他模块。
```

---

## 七、小结

Testcontainers 可以让 Spring Boot 集成测试摆脱本机数据库依赖，是团队级测试体系的重要基础。Claude Code 很适合基于 Controller 生成测试初稿，但开发者需要检查断言是否真正覆盖业务语义。下一篇将总结 Spring Boot 开发中最实用的 Claude Code 使用技巧。

---

## 系列导航

- [返回系列目录]({{ '/spring-boot-series/' | relative_url }})
- 上一篇：[修复 @Transactional 事务失效与循环依赖]({% post_url 2026-06-20-Claude-Code-Spring-Boot-深度实战-05-事务与循环依赖调试 %})
- 下一篇：[最佳实践与高效技巧]({% post_url 2026-06-20-Claude-Code-Spring-Boot-深度实战-07-最佳实践 %})

---
