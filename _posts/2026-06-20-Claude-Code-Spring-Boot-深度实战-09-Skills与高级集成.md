---
title: "Claude Code 在 Spring Boot 项目中的深度实战 09：自定义 Skills 与高级集成"
date: 2026-06-20
categories:
  - Java
  - Spring Boot
  - Claude Code
tags:
  - Skills
  - MCP
  - 高级集成
  - 团队规范
---

## 一、为什么需要自定义 Skills

当团队频繁使用 Claude Code 处理类似任务时，仅靠每次输入长提示词会变得低效。例如基于 Entity 生成 CRUD、为 Controller 生成 Testcontainers 测试、检查事务失效、生成 DTO 映射代码，这些任务都有固定流程和规范。**Skills** 的作用就是把这些高频任务沉淀成可复用的能力。

一个 Skill 本质上是一个目录，里面有 `SKILL.md`，用于描述技能适用场景、执行步骤、输入输出约束和注意事项。Claude Code 在任务匹配时可以加载对应 Skill，从而减少重复提示。

---

## 二、Spring Boot CRUD Skill 示例

目录结构：

```text
.claude/skills/spring-boot-crud/
└── SKILL.md
```

`SKILL.md` 内容：

```markdown
---
name: spring-boot-crud
description: Generate Spring Boot 3.x CRUD layers from a JPA entity using project conventions.
---

## 适用场景

当用户要求基于 JPA Entity 生成 Repository、Service、Controller、DTO 或测试时使用。

## 输入要求

用户必须提供 Entity 文件路径，例如：
`src/main/java/com/yourcompany/demo/entity/User.java`

## 生成规则

1. Repository 继承 JpaRepository<Entity, Long>。
2. Service 接口命名为 I{Entity}Service。
3. Service 实现类命名为 {Entity}ServiceImpl。
4. Controller 使用 @RestController。
5. Controller 路径为 /api/{plural-name}。
6. 所有 Controller 返回 Result<T>。
7. 使用构造器注入。
8. 不创建重复 Result 类。
9. 不修改无关模块。

## 验证步骤

生成后提示用户运行：

```bash
mvn clean compile
mvn test
```
```

使用方式：

```text
使用 spring-boot-crud 技能，基于 User.java 生成 CRUD。
```

---

## 三、事务诊断 Skill 示例

目录：

```text
.claude/skills/spring-transaction-debug/
└── SKILL.md
```

内容：

```markdown
---
name: spring-transaction-debug
description: Diagnose Spring @Transactional failures and circular dependencies.
---

## 适用场景

当用户反馈 @Transactional 未生效、数据未回滚、循环依赖、Bean 创建失败时使用。

## 诊断清单

1. 是否存在同类内部调用。
2. 事务方法是否为 public。
3. 异常是否被 catch 后吞掉。
4. 是否缺少 rollbackFor。
5. 当前类是否由 Spring 管理。
6. 是否存在构造器循环依赖。
7. 是否应拆分 Service 职责。

## 输出格式

- 问题结论
- 根本原因
- 最小修复方案
- 长期优化方案
- 修改前后代码对比
```

这种 Skill 可以帮助团队统一排查事务问题，避免每次都从零描述。

---

## 四、测试生成 Skill 示例

测试 Skill 可以专门约束 Testcontainers：

```markdown
---
name: spring-boot-integration-test
description: Generate Spring Boot 3.x integration tests with Testcontainers.
---

## 规则

1. 使用 @SpringBootTest(webEnvironment = RANDOM_PORT)。
2. 使用 Testcontainers 启动 MySQL 8.0。
3. 使用 @DynamicPropertySource 注入数据源。
4. 使用 RestAssured 或 TestRestTemplate。
5. 每个测试前清理数据。
6. 断言 HTTP 状态码、Result.code、Result.msg、Result.data。
7. 不依赖本机数据库。
```

调用：

```text
使用 spring-boot-integration-test 技能，为 UserController 生成集成测试。
```

---

## 五、与 MCP 的集成思路

MCP，即 Model Context Protocol，可以把外部系统能力接入 AI 工作流。例如 GitHub、任务系统、文档系统、数据库元数据服务等。对 Spring Boot 团队来说，常见集成方向包括：

- 读取 GitHub Issue，根据需求生成代码。
- 查询接口文档，生成 Controller。
- 读取数据库表结构，生成 Entity。
- 对接 CI 平台，分析失败流水线。

在实际团队中，可以把 Claude Code 与 GitHub 工作流结合：

```text
Issue 描述需求
        ↓
Claude Code 生成实现方案
        ↓
开发者确认后生成代码
        ↓
运行测试
        ↓
提交 PR
```

需要注意，任何涉及外部系统写操作的集成都应有人工确认，例如创建 PR、修改 Issue 状态、删除远程分支等。

---

## 六、团队级 CLAUDE.md 与 Skills 配合

`CLAUDE.md` 适合写全局规范，例如：

```markdown
- Java 版本为 17。
- Spring Boot 版本为 3.x。
- Controller 返回 Result<T>。
- Service 使用接口 + 实现类。
- Repository 继承 JpaRepository。
```

Skills 适合写任务流程，例如：

```markdown
- 如何从 Entity 生成 CRUD。
- 如何生成集成测试。
- 如何诊断事务问题。
- 如何审查 Controller 安全风险。
```

两者配合后，Claude Code 既知道项目规范，也知道具体任务怎么做。

---

## 七、高级提示词：生成 CRUD + 测试

```text
请使用 spring-boot-crud 和 spring-boot-integration-test 两个技能，基于 User 实体生成完整 CRUD 和集成测试。

要求：
1. 先读取 CLAUDE.md。
2. 只处理 User 模块。
3. 生成 Repository、IUserService、UserServiceImpl、UserController。
4. 生成 UserControllerIntegrationTest。
5. 测试使用 Testcontainers MySQL 8.0。
6. 所有接口返回 Result<T>。
7. 生成后列出文件清单和运行命令。
```

这样的提示词把通用规则交给 `CLAUDE.md`，把任务细节交给 Skills，开发者只需要描述实体和目标。

---

## 八、小结

当个人使用 Claude Code 时，提示词足够；当团队反复处理相似任务时，必须沉淀 `CLAUDE.md` 和 Skills。对 Spring Boot 团队而言，最值得建设的 Skills 包括 CRUD 生成、事务诊断、集成测试生成、Controller 安全审查和 JPA 查询优化。下一篇将总结完整学习路线与进阶方向。

---
