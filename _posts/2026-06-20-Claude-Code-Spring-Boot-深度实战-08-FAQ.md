---
title: "Claude Code 在 Spring Boot 项目中的深度实战 08：常见问题 FAQ"
date: 2026-06-20
categories:
  - Java
  - Spring Boot
  - Claude Code
tags:
  - FAQ
  - Spring Boot 调试
  - Maven
  - JPA
---

## 一、FAQ 的价值

Spring Boot 开发者使用 Claude Code 时，最常遇到的问题不是“它不会写代码”，而是生成结果不符合项目规范、上下文读取过多、测试跑不起来、事务分析不准确、依赖版本冲突等。本文整理至少 6 个高频问题，每个包含问题描述、根本原因和可执行解决方案。

---

## 二、问题一：Claude Code 生成的 Service 命名不符合规范

### 问题描述

项目要求接口叫 `IUserService`，实现类叫 `UserServiceImpl`，但 Claude Code 生成了 `UserService` 和 `UserService` 实现类，导致命名混乱。

### 根本原因

项目规范没有写入 `CLAUDE.md`，或者提示词中没有明确命名规则。Claude Code 会按常见 Spring 项目习惯生成代码。

### 解决方案

在 `CLAUDE.md` 中写入：

```markdown
## Service 命名规范

- Service 接口命名格式为 I{Entity}Service。
- Service 实现类命名格式为 {Entity}ServiceImpl。
- Service 实现类放在 service.impl 包。
```

提示词中再次强调：

```text
请严格按照 CLAUDE.md 中的 Service 命名规范生成代码，不要创建 UserService 接口。
```

---

## 三、问题二：Claude Code 修改了不相关模块

### 问题描述

只想修改 User 模块，但 Role、Order 或 Product 模块也被改了。

### 根本原因

提示词范围不清晰，Claude Code 为了寻找参考实现读取了其他模块，并把其他模块也纳入了修改范围。

### 解决方案

使用范围限定提示词：

```text
只处理 User 模块。允许读取和修改的文件仅限：
- UserController.java
- IUserService.java
- UserServiceImpl.java
- UserRepository.java
- User.java
不要读取或修改其他模块。
```

如果需要参考其他模块，只允许读取不允许修改：

```text
可以只读 OrderController 作为风格参考，但禁止修改 Order 模块任何文件。
```

---

## 四、问题三：生成的 Controller 没有使用 Result<T>

### 问题描述

Controller 返回了 `User`、`List<User>` 或 `ResponseEntity<User>`，没有使用项目统一响应结构。

### 根本原因

Claude Code 没有识别到项目已有 `Result<T>`，或者统一响应规范没有写清楚。

### 解决方案

先让它读取：

```text
请先读取 src/main/java/com/yourcompany/demo/common/Result.java，之后所有 Controller 接口必须返回 Result<T>。
```

示例要求：

```java
@GetMapping("/{id}")
public Result<User> findById(@PathVariable Long id) {
    return Result.success(userService.findById(id));
}
```

---

## 五、问题四：@Transactional 事务没有生效

### 问题描述

方法标注了 `@Transactional`，但异常发生后数据库没有回滚。

### 根本原因

常见原因包括：

1. 同类内部方法调用绕过 Spring AOP 代理。
2. 方法不是 `public`。
3. 异常被 catch 后没有继续抛出。
4. 抛出的是受检异常，但没有配置 `rollbackFor`。
5. 当前类没有被 Spring 管理。

### 解决方案

提示 Claude Code：

```text
请检查该事务方法是否存在同类内部调用、异常吞掉、非 public 方法、rollbackFor 缺失等问题。先分析原因，再给最小修复方案。
```

推荐写法：

```java
@Transactional(rollbackFor = Exception.class)
public void batchUpdate(List<User> users) {
    // 写操作
}
```

如果存在自调用，可通过拆分 Service 或获取代理对象解决。

---

## 六、问题五：Testcontainers 测试无法启动 MySQL

### 问题描述

执行测试时报错：

```text
Could not find a valid Docker environment
```

### 根本原因

Testcontainers 依赖 Docker。Docker Desktop 没有启动，或者 CI 环境没有配置 Docker。

### 解决方案

本地先验证：

```bash
docker version
docker ps
```

然后执行：

```bash
mvn -Dtest=UserControllerIntegrationTest test
```

GitHub Actions 中需要使用支持 Docker 的 runner，通常 `ubuntu-latest` 默认可用。

---

## 七、问题六：Claude Code 生成的测试只断言状态码

### 问题描述

测试中只有：

```java
.then().statusCode(200);
```

业务字段没有断言。

### 根本原因

提示词没有要求断言业务语义。AI 默认可能生成最小可运行测试。

### 解决方案

明确要求：

```text
测试必须断言：
1. HTTP 状态码。
2. Result.code。
3. Result.msg。
4. Result.data 中的核心字段。
5. 数据库最终状态。
```

示例：

```java
.then()
    .statusCode(200)
    .body("code", equalTo(200))
    .body("data.username", equalTo("zhangsan"));
```

---

## 八、问题七：Claude Code 推荐了过度设计

### 问题描述

只想新增一个接口，结果它建议引入 MapStruct、事件总线、领域服务、缓存和复杂异常体系。

### 根本原因

提示词没有限制“最小改动”。模型为了给出完整架构方案，可能倾向于扩展设计。

### 解决方案

使用约束：

```text
请采用最小修改方案，不引入新依赖，不新增抽象层，不修改数据库结构，只完成当前接口。
```

对于复杂建议，可以回复：

```text
这些属于后续优化。本次只实现 UserController 的查询接口。
```

---

## 九、问题八：Maven 或 Gradle 命令不适合当前项目

### 问题描述

Claude Code 给出了 Maven 命令，但项目使用 Gradle；或者给出的单测命令不正确。

### 根本原因

它没有先读取构建文件，或者项目同时存在多个构建脚本。

### 解决方案

提示：

```text
请先读取 pom.xml 或 build.gradle，判断项目使用 Maven 还是 Gradle，再给出构建和测试命令。
```

常用命令：

```bash
mvn test
mvn -Dtest=UserServiceTest test
gradle test
gradle test --tests UserServiceTest
```

---

## 十、小结

高频问题大多可以通过三种方式避免：第一，把规范写进 `CLAUDE.md`；第二，提示词明确任务边界；第三，生成后必须用编译和测试验证。Claude Code 是强大的协作工具，但最终质量仍取决于开发者是否给出清晰约束并认真 review。

---
