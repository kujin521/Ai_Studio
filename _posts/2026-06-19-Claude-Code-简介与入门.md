---
title: "Claude Code 简介与入门"
date: 2026-06-19
categories:
  - AI
  - Claude Code
tags:
  - Claude Code
  - 入门
  - AI 编程
---

## 一、什么是 Claude Code？

**Claude Code** 是 Anthropic 公司推出的**终端交互式 AI 编程助手**。简单说，它是在你的终端（命令行）里运行的一个 AI 程序员——你告诉它你想做什么，它就能帮你读写代码、分析项目、回答问题。

### 它和你是什么关系？

```
你：在终端输入需求（中文/英文都可以）
       │
       ▼
Claude Code：理解你的意图 → 分析你的代码 → 执行操作（读写文件、运行命令）
       │
       ▼
你：检查结果 → 提出修改意见 → 循环直到满意
```

### 核心特点

- **运行在终端** — 不依赖 IDE 插件，任何编辑器都能配合使用
- **全项目感知** — 能理解整个项目的结构和逻辑
- **工具调用** — 能直接读写文件、搜索代码、运行命令、操作 Git
- **多模型支持** — 默认使用 Claude，也可切换到其他模型

---

## 二、安装 Claude Code

### 前提条件

| 要求 | 说明 |
|------|------|
| Node.js | 版本 18.0 或更高 |
| npm | 一般随 Node.js 一起安装 |
| 网络 | 能正常访问 Anthropic API |

### 安装步骤

```bash
# 1. 全局安装
npm install -g @anthropic-ai/claude-code

# 2. 验证安装
claude --version

# 3. 登录（需要 Anthropic API Key）
claude login
```

### 在项目中启动

```bash
# 进入你的项目目录
cd E:/Code/Demo/Ai_Studio

# 启动 Claude Code
claude
```

首次启动会进行项目分析，Claude 会读取项目结构并创建 `.claude/` 目录存放配置信息。

---

## 三、第一次对话

启动后你会看到交互式终端界面，直接输入你的需求即可。

### 示例对话

```
你：这个项目是做什么的？帮我看看项目结构

Claude：我来看看……
  ├── 项目说明.md
  ├── _config.yml
  ├── _posts/
  ├── index.md
  └── ...

这个项目是一个基于 Jekyll 的个人博客站点，用来展示 AI 学习笔记。
```

```
你：帮我在 _posts 目录创建一个新的文章模板

Claude：好的，我来创建……

[创建了 _posts/template.md 文件]
```

---

## 四、两种使用模式

### 模式一：交互式对话（推荐新手）

```bash
claude
# 进入对话模式，像聊天一样交流
```

适合：探索式开发、有问题就问、逐步完成任务。

### 模式二：一次性指令

```bash
claude -p "解释一下项目说明.md的内容"
```

适合：快速问答、CI/CD 集成、脚本调用。

---

## 五、基础命令速查

| 命令 | 作用 | 示例 |
|------|------|------|
| `/help` | 查看帮助 | `/help` |
| `/clear` | 清除对话历史 | `/clear` |
| `/cost` | 查看当前会话使用量 | `/cost` |
| `claude -p "xxx"` | 一次性提问 | `claude -p "分析这个文件"` |
| `claude --resume` | 恢复上次会话 | `claude --resume` |

---

## 六、新手注意事项

### ✅ 做

- 每次只提一个明确的需求：_"帮我创建一个 Spring Boot 的 HelloController"_
- 让 Claude 先理解现有代码再修改
- 检查 Claude 生成的代码后再使用

### ❌ 不要做

- 一次提十几个需求（会超出上下文限制）
- 让 Claude 执行你不理解的命令
- 在不清楚的情况下让 Claude 联网获取信息

---

## 七、小结

```
知识点清单：
☑ 知道 Claude Code 是什么
☑ 完成了安装和登录
☑ 进行了第一次对话
☑ 了解交互模式和一次性指令的区别
☑ 知道常用命令
```

下一步 → 学习 Claude Code 的核心功能
