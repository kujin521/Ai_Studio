---
title: "Claude Code 在 Spring Boot 项目中的深度实战 04：从 Entity 生成 CRUD REST API"
date: 2026-06-20
categories:
  - Java
  - Spring Boot
  - Claude Code
tags:
  - CRUD
  - Spring Data JPA
  - REST API
  - Result
---

## 一、场景说明

本案例面向入门级 Spring Boot 开发者，目标是在 5 分钟内基于已有 `User.java` 实体类生成完整 CRUD REST API。这个案例非常适合第一次在真实 Spring Boot 项目中使用 Claude Code，因为它覆盖了最典型的后端开发闭环：Entity、Repository、Service、Controller 和统一响应对象。

假设项目已有实体类：

```text
src/main/java/com/yourcompany/demo/entity/User.java
```

内容如下：

```java
package com.yourcompany.demo.entity;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "sys_user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 64)
    private String username;

    @Column(nullable = false, length = 128)
    private String email;

    @Column(nullable = false)
    private Boolean enabled;
}
```

项目中已经存在统一返回对象：

```text
src/main/java/com/yourcompany/demo/common/Result.java
```

如果没有，应先建立统一响应规范，不要让每个 Controller 自己定义返回结构。

---

## 二、完整提示词指令

在项目根目录启动 Claude Code：

```bash
claude
```

输入以下提示词：

```text
请基于 src/main/java/com/yourcompany/demo/entity/User.java 生成完整 CRUD REST API。

请严格遵守以下要求：
1. 只处理 User 模块，不修改 Role、Order 或其他模块。
2. Repository 放在 repository 包，命名为 UserRepository，继承 JpaRepository<User, Long>。
3. Service 接口放在 service 包，命名为 IUserService。
4. Service 实现类放在 service.impl 包，命名为 UserServiceImpl，并添加 @Service。
5. Controller 放在 controller 包，命名为 UserController，并添加 @RestController。
6. Controller 路径为 /api/users。
7. 所有接口统一返回 Result<T>。
8. 包含 save、findById、findAll、update、deleteById 五个方法。
9. 使用构造器注入，不要使用 @Autowired 字段注入。
10. 不要重复创建 Result 类；如果需要，请先读取已有 common/Result.java。
11. 生成后请列出新增文件，并说明每个接口对应的 HTTP 方法和路径。
```

这个提示词的关键点是**限制范围**和**明确命名规则**。如果只说“帮我生成 CRUD”，Claude Code 可能会根据通用习惯生成 `UserService` 而不是 `IUserService`，也可能返回 `ResponseEntity` 而不是 `Result<T>`。

---

## 三、Claude Code 终端操作截图描述

实际终端中一般会出现类似过程：

```text
Claude Code 正在读取：
- src/main/java/com/yourcompany/demo/entity/User.java
- src/main/java/com/yourcompany/demo/common/Result.java

Claude Code 准备创建：
- src/main/java/com/yourcompany/demo/repository/UserRepository.java
- src/main/java/com/yourcompany/demo/service/IUserService.java
- src/main/java/com/yourcompany/demo/service/impl/UserServiceImpl.java
- src/main/java/com/yourcompany/demo/controller/UserController.java

Claude Code 请求写入文件权限：Allow? [y/N]
```

如果你确认文件路径和范围正确，可以批准。生成后建议要求它不要立即提交 Git，而是先总结变更：

```text
请先不要提交 Git，先列出新增文件和接口清单。
```

---

## 四、生成后的 Repository 示例

```java
package com.yourcompany.demo.repository;

import com.yourcompany.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
```

Repository 保持简单，不要在一开始就加入复杂查询。复杂查询应在业务需求明确后再增加。

---

## 五、生成后的 Service 接口

```java
package com.yourcompany.demo.service;

import com.yourcompany.demo.entity.User;
import java.util.List;

public interface IUserService {

    User save(User user);

    User findById(Long id);

    List<User> findAll();

    User update(Long id, User user);

    void deleteById(Long id);
}
```

如果项目更严格，应使用 DTO 替代直接暴露 Entity。例如 `UserCreateRequest`、`UserUpdateRequest`、`UserResponse`。本案例为了聚焦 Claude Code CRUD 生成流程，先使用 Entity 演示。

---

## 六、生成后的 Service 实现

```java
package com.yourcompany.demo.service.impl;

import com.yourcompany.demo.entity.User;
import com.yourcompany.demo.repository.UserRepository;
import com.yourcompany.demo.service.IUserService;
import jakarta.persistence.EntityNotFoundException;
import java.util.List;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class UserServiceImpl implements IUserService {

    private final UserRepository userRepository;

    @Override
    @Transactional(rollbackFor = Exception.class)
    public User save(User user) {
        return userRepository.save(user);
    }

    @Override
    @Transactional(readOnly = true)
    public User findById(Long id) {
        return userRepository.findById(id)
                .orElseThrow(() -> new EntityNotFoundException("用户不存在，id=" + id));
    }

    @Override
    @Transactional(readOnly = true)
    public List<User> findAll() {
        return userRepository.findAll();
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public User update(Long id, User user) {
        User existUser = findById(id);
        existUser.setUsername(user.getUsername());
        existUser.setEmail(user.getEmail());
        existUser.setEnabled(user.getEnabled());
        return userRepository.save(existUser);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void deleteById(Long id) {
        User existUser = findById(id);
        userRepository.delete(existUser);
    }
}
```

这里有一个细节：`update` 方法内部调用 `findById` 是同类内部调用，不会重新经过代理，但因为外层 `update` 自身已经有事务，所以这里没有事务失效问题。如果是一个无事务方法调用同类中的事务方法，就需要特别注意。

---

## 七、生成后的 Controller 示例

```java
package com.yourcompany.demo.controller;

import com.yourcompany.demo.common.Result;
import com.yourcompany.demo.entity.User;
import com.yourcompany.demo.service.IUserService;
import java.util.List;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/users")
public class UserController {

    private final IUserService userService;

    @PostMapping
    public Result<User> save(@RequestBody User user) {
        return Result.success(userService.save(user));
    }

    @GetMapping("/{id}")
    public Result<User> findById(@PathVariable Long id) {
        return Result.success(userService.findById(id));
    }

    @GetMapping
    public Result<List<User>> findAll() {
        return Result.success(userService.findAll());
    }

    @PutMapping("/{id}")
    public Result<User> update(@PathVariable Long id, @RequestBody User user) {
        return Result.success(userService.update(id, user));
    }

    @DeleteMapping("/{id}")
    public Result<Void> deleteById(@PathVariable Long id) {
        userService.deleteById(id);
        return Result.success(null);
    }
}
```

---

## 八、验证命令

Maven 项目：

```bash
mvn clean compile
mvn test
```

Gradle 项目：

```bash
gradle clean compileJava
gradle test
```

如果编译失败，把完整错误贴给 Claude Code：

```text
下面是 mvn compile 的完整错误日志，请只分析 User 模块相关问题并修复，不要修改其他模块：

[粘贴日志]
```

---

## 九、小结

从 Entity 生成 CRUD 是 Claude Code 在 Spring Boot 项目中最容易产生价值的场景。关键不是让它“随便生成”，而是通过提示词锁定分层、命名、返回类型、事务边界和文件范围。下一篇将进入进阶案例：修复 `@Transactional` 事务失效与循环依赖问题。

---
