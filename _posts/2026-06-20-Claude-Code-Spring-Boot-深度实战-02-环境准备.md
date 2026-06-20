---
title: "Claude Code 在 Spring Boot 项目中的深度实战 02：环境准备与 CLAUDE.md 配置"
date: 2026-06-20
categories:
  - Java
  - Spring Boot
  - Claude Code
tags:
  - CLAUDE.md
  - Maven
  - Gradle
  - Spring Boot 配置
---

## 一、为什么 Spring Boot 项目必须重视 CLAUDE.md

`CLAUDE.md` 是 Claude Code 在当前项目中的长期指令文件。对于 Spring Boot 项目，它相当于项目的“开发宪法”：包结构怎么分层、Controller 如何返回、Service 命名规则是什么、是否使用 Lombok、构建命令是什么、测试命令如何运行，都应该写在这个文件中。

如果没有 `CLAUDE.md`，Claude Code 只能通过读取项目文件推断规范。它可能生成 `UserService`，而你的项目实际约定是 `IUserService`；它可能直接返回 `User`，而你的项目要求统一返回 `Result<T>`；它也可能使用字段注入，而你的团队要求构造器注入。把这些规范提前写清楚，能显著减少返工。

---

## 二、推荐的 Spring Boot 项目结构

一个典型 Spring Boot 3.x 项目可以采用如下结构：

```text
src/main/java/com/yourcompany/demo/
├── DemoApplication.java
├── controller/
├── service/
│   └── impl/
├── repository/
├── entity/
├── dto/
├── vo/
├── common/
│   ├── Result.java
│   └── ResultCode.java
├── exception/
└── config/
```

其中：

- `controller`：只负责 HTTP 参数接收、参数校验和响应封装。
- `service`：定义业务接口，例如 `IUserService`。
- `service.impl`：实现业务逻辑，例如 `UserServiceImpl`。
- `repository`：继承 Spring Data JPA 的 Repository。
- `entity`：JPA 实体类，只表达持久化结构。
- `dto`：请求对象，例如 `UserCreateRequest`、`UserUpdateRequest`。
- `vo`：响应视图对象，例如 `UserResponse`。
- `common`：统一返回对象、常量、枚举等。

---

## 三、CLAUDE.md 模板

可以在 Spring Boot 项目根目录创建如下文件：

```markdown
# CLAUDE.md

This file provides guidance to Claude Code when working with this Spring Boot repository.

## 项目技术栈

- Java 17
- Spring Boot 3.x
- Spring Data JPA
- MySQL 8.0
- Maven 或 Gradle
- JUnit 5
- Testcontainers
- Lombok

## 构建与测试命令

### Maven

```bash
mvn clean compile
mvn test
mvn -Dtest=UserControllerIntegrationTest test
mvn spring-boot:run
```

### Gradle

```bash
gradle clean build
gradle test
gradle test --tests UserControllerIntegrationTest
gradle bootRun
```

## 分层包结构规范

- Controller 放在 `controller` 包。
- Service 接口放在 `service` 包，命名格式为 `IxxxService`。
- Service 实现放在 `service.impl` 包，命名格式为 `xxxServiceImpl`。
- Repository 放在 `repository` 包，继承 `JpaRepository<Entity, Long>`。
- Entity 放在 `entity` 包。
- DTO 放在 `dto` 包。
- 统一返回对象放在 `common` 包。

## 编码规范

- 使用构造器注入，不使用字段注入。
- 使用 Lombok 的 `@Getter`、`@Setter`、`@Builder`、`@NoArgsConstructor`、`@AllArgsConstructor`。
- Controller 不写复杂业务逻辑。
- Service 层负责事务边界。
- 所有 Controller 返回 `Result<T>`。
- 不返回 `null` 表示失败，使用业务异常或明确的失败响应。

## 统一响应格式

`Result<T>` 包含：

- `code`
- `msg`
- `data`

成功响应使用 `Result.success(data)`。
失败响应使用 `Result.fail(code, msg)`。

## application.yml 规范

- `application.yml` 存放公共配置。
- `application-dev.yml` 存放开发环境配置。
- `application-test.yml` 存放测试环境配置。
- `application-prod.yml` 存放生产环境配置。
- 不要把密码、Token、云服务密钥写入 Git。
```

这个模板可以作为起点，再根据团队实际规范扩展。

---

## 四、统一返回对象示例

建议在项目中提供基础返回结构，便于 Claude Code 生成 Controller 时保持一致。

```java
package com.yourcompany.demo.common;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Result<T> {

    private Integer code;

    private String msg;

    private T data;

    public static <T> Result<T> success(T data) {
        return new Result<>(200, "success", data);
    }

    public static <T> Result<T> fail(Integer code, String msg) {
        return new Result<>(code, msg, null);
    }
}
```

如果项目已有自己的 `Result`，应以现有实现为准，不要让 Claude Code 新建重复类。

---

## 五、application.yml 多环境配置

推荐结构如下：

```yaml
spring:
  profiles:
    active: dev
  application:
    name: claude-code-spring-demo

server:
  port: 8080
```

开发环境：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/demo_dev?useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: root
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

测试环境：

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
```

生产环境中不建议使用 `ddl-auto: update`，应使用 Flyway 或 Liquibase 管理数据库变更。

---

## 六、验证 Claude Code 是否识别项目

进入项目根目录后启动：

```bash
claude
```

输入：

```text
请分析这个 Spring Boot 项目的技术栈、包结构、构建工具和统一返回格式。只读取 CLAUDE.md、pom.xml 或 build.gradle、src/main/java 下的主启动类和 common/Result.java。
```

理想结果应该包含：

- 识别 Java 17 / Spring Boot 3.x。
- 识别 Maven 或 Gradle。
- 识别 Controller / Service / Repository 分层。
- 识别 `Result<T>` 响应约定。
- 不主动扫描无关模块。

---

## 七、小结

环境准备的核心不是安装工具，而是把项目规范写清楚。`CLAUDE.md` 越明确，Claude Code 生成的代码越接近团队标准。对于 Spring Boot 项目，必须提前约束构建命令、包结构、Lombok 规则、统一返回格式和多环境配置。下一篇将解释 Agent Loop、Plan Mode、上下文管理和 Skills 等核心概念。

---
