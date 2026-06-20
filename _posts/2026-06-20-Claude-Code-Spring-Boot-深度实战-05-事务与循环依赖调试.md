---
title: "Claude Code 在 Spring Boot 项目中的深度实战 05：修复 @Transactional 事务失效与循环依赖"
date: 2026-06-20
categories:
  - Java
  - Spring Boot
  - Claude Code
tags:
  - Transactional
  - 循环依赖
  - 调试
  - AOP
---

## 一、案例背景

本案例是进阶级调试场景，预计耗时 15 分钟。问题发生在 `UserServiceImpl` 中：`batchUpdate` 方法已经标注 `@Transactional`，但实际运行时部分数据更新成功、部分失败后没有回滚。同时，应用启动日志还提示 `UserService` 与 `RoleService` 之间存在循环依赖。

这类问题在 Spring Boot 项目中非常常见，根本原因通常不是注解写错，而是 Spring AOP 代理机制、Bean 注入关系和事务边界设计不合理。Claude Code 的价值在于，它可以同时阅读错误日志、Service 实现类和 Bean 依赖关系，帮助你形成完整判断。

---

## 二、修复前代码示例

`UserServiceImpl`：

```java
package com.yourcompany.demo.service.impl;

import com.yourcompany.demo.entity.User;
import com.yourcompany.demo.repository.UserRepository;
import com.yourcompany.demo.service.IRoleService;
import com.yourcompany.demo.service.IUserService;
import java.util.List;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class UserServiceImpl implements IUserService {

    private final UserRepository userRepository;

    private final IRoleService roleService;

    @Override
    public void updateUsers(List<User> users) {
        batchUpdate(users);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void batchUpdate(List<User> users) {
        for (User user : users) {
            userRepository.save(user);
            roleService.refreshUserRole(user.getId());
        }
    }
}
```

`RoleServiceImpl`：

```java
package com.yourcompany.demo.service.impl;

import com.yourcompany.demo.service.IRoleService;
import com.yourcompany.demo.service.IUserService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class RoleServiceImpl implements IRoleService {

    private final IUserService userService;

    @Override
    public void refreshUserRole(Long userId) {
        userService.findById(userId);
    }
}
```

这里有两个问题：

1. `updateUsers` 内部直接调用同类方法 `batchUpdate`，绕过了 Spring AOP 代理，导致事务可能不生效。
2. `UserServiceImpl` 依赖 `IRoleService`，`RoleServiceImpl` 又依赖 `IUserService`，形成循环依赖。

---

## 三、如何把错误日志交给 Claude Code

启动失败或警告日志可能类似：

```text
The dependencies of some of the beans in the application context form a cycle:

userServiceImpl defined in file [UserServiceImpl.class]
┌─────┐
|  roleServiceImpl defined in file [RoleServiceImpl.class]
↑     ↓
|  userServiceImpl
└─────┘
```

事务异常可能表现为：

```text
batchUpdate executed but transaction rollback did not happen when RoleService threw RuntimeException
```

完整诊断提示词：

```text
下面是 Spring Boot 启动日志和事务异常现象，请帮我诊断并修复。

问题现象：
1. UserServiceImpl.batchUpdate 标注了 @Transactional，但 roleService.refreshUserRole 抛出 RuntimeException 后，前面保存的用户没有回滚。
2. 应用启动日志提示 UserServiceImpl 和 RoleServiceImpl 存在循环依赖。

请只读取并分析以下文件：
- src/main/java/com/yourcompany/demo/service/IUserService.java
- src/main/java/com/yourcompany/demo/service/IRoleService.java
- src/main/java/com/yourcompany/demo/service/impl/UserServiceImpl.java
- src/main/java/com/yourcompany/demo/service/impl/RoleServiceImpl.java

要求：
1. 先解释事务失效的根本原因。
2. 再解释循环依赖的根本原因。
3. 给出最小修改方案。
4. 不要修改 Controller 和 Repository。
5. 修复后给出修改前后代码对比。

错误日志如下：
[在这里粘贴完整日志]
```

---

## 四、Claude Code 应该给出的分析结论

合理分析应包含：

- Spring 的 `@Transactional` 基于代理生效。
- 同类内部方法调用不会经过代理对象。
- `updateUsers` 调用 `this.batchUpdate(users)`，事务拦截器不会触发。
- 构造器注入形成强依赖闭环，导致循环依赖。
- 解决方案要么拆分职责，要么延迟获取其中一个 Bean。

其中最推荐的长期方案是拆分业务边界，避免 User 和 Role 互相依赖。但在最小修改场景下，可以使用 `ApplicationContext.getBean` 获取代理对象，或者对其中一个依赖使用 `@Lazy`。

---

## 五、修复方案一：使用 ApplicationContext 获取代理对象

修复后 `UserServiceImpl`：

```java
package com.yourcompany.demo.service.impl;

import com.yourcompany.demo.entity.User;
import com.yourcompany.demo.repository.UserRepository;
import com.yourcompany.demo.service.IRoleService;
import com.yourcompany.demo.service.IUserService;
import java.util.List;
import lombok.RequiredArgsConstructor;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class UserServiceImpl implements IUserService {

    private final UserRepository userRepository;

    private final IRoleService roleService;

    private final ApplicationContext applicationContext;

    @Override
    public void updateUsers(List<User> users) {
        IUserService userServiceProxy = applicationContext.getBean(IUserService.class);
        userServiceProxy.batchUpdate(users);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void batchUpdate(List<User> users) {
        for (User user : users) {
            userRepository.save(user);
            roleService.refreshUserRole(user.getId());
        }
    }
}
```

这个方案解决事务自调用问题，因为 `updateUsers` 获取的是 Spring 容器中的代理对象，而不是 `this`。

但它没有完全解决循环依赖。如果 `RoleServiceImpl` 仍然构造器注入 `IUserService`，仍可能存在启动问题。

---

## 六、修复方案二：使用 @Lazy 打破循环依赖

`RoleServiceImpl` 可以改为：

```java
package com.yourcompany.demo.service.impl;

import com.yourcompany.demo.service.IRoleService;
import com.yourcompany.demo.service.IUserService;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Service;

@Service
public class RoleServiceImpl implements IRoleService {

    private final IUserService userService;

    public RoleServiceImpl(@Lazy IUserService userService) {
        this.userService = userService;
    }

    @Override
    public void refreshUserRole(Long userId) {
        userService.findById(userId);
    }
}
```

`@Lazy` 会延迟注入代理对象，从而打破启动时的强依赖环。

不过要注意：`@Lazy` 是工程上的折中方案，不是架构上的根治方案。如果两个 Service 长期互相调用，说明职责边界可能需要重新设计。

---

## 七、修复前后对比

修复前：

```java
@Override
public void updateUsers(List<User> users) {
    batchUpdate(users);
}
```

修复后：

```java
@Override
public void updateUsers(List<User> users) {
    IUserService userServiceProxy = applicationContext.getBean(IUserService.class);
    userServiceProxy.batchUpdate(users);
}
```

修复前：

```java
@RequiredArgsConstructor
public class RoleServiceImpl implements IRoleService {

    private final IUserService userService;
}
```

修复后：

```java
public class RoleServiceImpl implements IRoleService {

    private final IUserService userService;

    public RoleServiceImpl(@Lazy IUserService userService) {
        this.userService = userService;
    }
}
```

---

## 八、验证命令

Maven：

```bash
mvn clean test
mvn -Dtest=UserServiceTransactionTest test
```

Gradle：

```bash
gradle clean test
gradle test --tests UserServiceTransactionTest
```

如果事务测试失败，应把完整测试日志继续交给 Claude Code：

```text
下面是 UserServiceTransactionTest 的失败日志。请判断是事务未生效、测试数据问题，还是异常没有抛出到事务边界外。
```

---

## 九、小结

`@Transactional` 失效和循环依赖不是简单的注解问题，而是 Spring AOP 代理机制和 Bean 设计问题。Claude Code 可以帮助你把日志、代码和框架机制串起来分析。最小修复可以使用 `ApplicationContext.getBean` 和 `@Lazy`，长期方案则应拆分 Service 职责，避免互相依赖。下一篇将进入综合案例：基于 Testcontainers 生成集成测试套件。

---
