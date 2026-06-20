---
title: "Claude Code 在 Spring Boot 项目中的深度实战 03：Agent Loop、Plan Mode 与上下文管理"
date: 2026-06-20
categories:
  - Java
  - Spring Boot
  - Claude Code
tags:
  - Agent Loop
  - Plan Mode
  - Context
  - Skills
---

## 一、用 Spring Boot 语境理解 Agent Loop

**Agent Loop** 可以理解为 Claude Code 的“观察、修改、验证、再修复”循环。传统聊天式 AI 往往只给出一段代码建议，而 Claude Code 可以在项目中读取文件、修改代码、执行命令、观察错误，再继续调整。

在 Spring Boot 项目中，一个典型 Agent Loop 可能是：

```text
1. 读取 UserController.java、IUserService.java、UserServiceImpl.java
2. 修改 UserServiceImpl 的业务逻辑
3. 执行 mvn compile
4. 发现编译错误：方法签名不匹配
5. 回到接口或实现类修正方法签名
6. 再次执行 mvn compile
7. 编译通过后总结修改内容
```

终端交互示例：

```text
你：请为 UserServiceImpl 增加 updateUser 方法，并确保能通过编译。

Claude Code：我会先查看 IUserService 和 UserServiceImpl 的现有方法签名，然后修改实现类。

Claude Code：需要运行 mvn compile 验证吗？
```

对应命令模板：

```bash
mvn compile
gradle compileJava
```

需要注意的是，Agent Loop 并不意味着可以无脑让 AI 自动改到通过。开发者应该关注它每一步要执行的命令，特别是涉及数据库、删除文件、推送代码等高风险操作时必须谨慎确认。

---

## 二、Plan Mode：重构前先要计划

**Plan Mode** 适合用于影响范围较大的任务，例如重构 Service 层事务边界、拆分 Controller、迁移字段、调整 DTO、添加缓存或统一异常处理。它的核心思想是：在真正改代码前，先让 Claude Code 读取必要文件并输出执行计划。

示例提示词：

```text
进入计划模式。我要重构 UserServiceImpl 的用户更新逻辑，目标是：
1. Controller 只接收 UserUpdateRequest。
2. Service 层负责校验用户是否存在。
3. Repository 只负责数据库访问。
4. 事务边界放在 Service 实现类方法上。
请先阅读 UserController、IUserService、UserServiceImpl、UserRepository 和 User 实体，然后给出修改计划，不要直接改代码。
```

一个合格计划应该包含：

- 涉及哪些文件。
- 每个文件改什么。
- 是否需要新增 DTO。
- 是否影响测试。
- 是否存在兼容风险。

如果计划中出现“顺手重构所有 Controller”之类的扩大范围行为，应要求收敛：

```text
只处理 User 模块，不要修改 Role、Order 或其他模块。
```

---

## 三、Context 管理：不要让 AI 看太多，也不要看太少

Spring Boot 项目通常文件很多，如果一次性让 Claude Code 分析整个项目，很容易造成上下文污染。正确方式是让它只读取当前任务相关文件。

### 1. 使用明确文件路径

```text
请只读取以下文件：
- src/main/java/com/yourcompany/demo/controller/UserController.java
- src/main/java/com/yourcompany/demo/service/IUserService.java
- src/main/java/com/yourcompany/demo/service/impl/UserServiceImpl.java
- src/main/java/com/yourcompany/demo/repository/UserRepository.java
```

### 2. 用 /clear 清理历史

当你完成一个模块后，建议清空上下文：

```bash
/clear
```

然后开启新的任务：

```text
现在开始处理 Role 模块。请不要参考上一轮 User 模块的实现细节，先读取 RoleController 和 RoleService。
```

### 3. 用 @文件名 精准指定上下文

在支持文件引用的环境中，可以使用类似：

```text
@src/main/java/com/yourcompany/demo/entity/User.java
请基于这个实体生成 Repository 和 Service。
```

这比“你看看项目里 User 相关代码”更可控。

---

## 四、CLAUDE.md 是项目宪法

`CLAUDE.md` 不应该写成泛泛而谈的说明，而应该写成能约束行为的规则。例如：

```markdown
## Service 规范

- Service 接口必须以 I 开头，例如 IUserService。
- Service 实现类必须放在 service.impl 包。
- Service 实现类方法上声明事务。
- 查询方法默认只读事务：@Transactional(readOnly = true)。
- 写操作使用 @Transactional(rollbackFor = Exception.class)。
```

这样，当你要求生成 Service 时，Claude Code 会更容易遵守规范。

不推荐写：

```markdown
请写高质量代码。
```

这类规则太抽象，无法约束具体行为。

---

## 五、Skills：把重复任务做成可复用能力

**Skills** 可以理解为“按需加载的专项说明”。如果团队经常生成 Spring Boot CRUD，可以创建一个 `spring-boot-crud` Skill，里面写清楚文件结构、命名规则、生成顺序、检查清单和示例提示词。

一个 Skill 的目录可以是：

```text
.claude/skills/spring-boot-crud/
└── SKILL.md
```

`SKILL.md` 示例：

```markdown
---
name: spring-boot-crud
description: Generate Spring Boot 3.x CRUD layers from a JPA entity using project conventions.
---

## 使用场景

当用户要求基于 Entity 生成 Controller、Service、Repository、DTO 时使用。

## 生成规则

1. Repository 继承 JpaRepository<Entity, Long>。
2. Service 接口命名为 I{Entity}Service。
3. Service 实现命名为 {Entity}ServiceImpl。
4. Controller 路径使用 /api/{kebab-case-plural}。
5. 所有接口返回 Result<T>。
6. 不创建重复的 Result 类。
7. 修改前先读取 Entity 和已有 common 包。
```

使用时可以说：

```text
使用 spring-boot-crud 技能，基于 User 实体生成完整 CRUD。
```

---

## 六、概念之间如何配合

实际开发中，这几个概念通常一起使用：

```text
CLAUDE.md 提供项目规则
        ↓
Plan Mode 先规划复杂改动
        ↓
Context 管理限制读取范围
        ↓
Agent Loop 修改并验证
        ↓
Skills 复用高频任务模板
```

例如生成 CRUD 时，可以直接使用 Skill；重构事务逻辑时，应先进入 Plan Mode；测试失败时，让 Agent Loop 根据日志继续修复；任务完成后用 `/clear` 开始新模块。

---

## 七、小结

Claude Code 的关键不是“能不能写代码”，而是能否在受控上下文中持续执行开发循环。对 Spring Boot 项目来说，必须掌握 Agent Loop、Plan Mode、Context 管理、CLAUDE.md 和 Skills。下一篇将进入第一个完整实操案例：从 JPA Entity 生成全套 CRUD REST API。

---
