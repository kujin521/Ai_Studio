# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概况

- **项目名称**: Ai_Studio
- **目的**: 学习 AI 技术，利用 AI 生成学习文档和总结，并通过 GitHub 构建个人博客页面
- **当前状态**: 项目初始化阶段，尚未有实际源代码
- **IDE**: IntelliJ IDEA
- **版本控制**: Git（master 分支，远程待配置）

## 项目结构

```
Ai_Studio/
├── 项目说明.md          # 项目描述文档
└── Ai_Studio.iml        # IntelliJ IDEA 模块配置
```

项目当前为空项目，尚无源代码和构建配置。

## 技术栈预期

根据 `.idea` 目录和 `.iml` 文件，这是一个 IntelliJ IDEA 项目。建议未来采用：

- **后端**: Java / Spring Boot（与用户全局开发规范一致）
- **构建工具**: Maven（pom.xml 待创建）
- **前端**: 博客页面可考虑 Vue / React 或纯静态页面
- **托管**: GitHub Pages 用于博客展示

## 开发命令

当前项目尚无构建系统，待 pom.xml 或 package.json 创建后补充。

## 架构说明

项目意图分为两层：

1. **AI 学习文档生成层** — 利用 AI 模型生成技术学习文档和总结
2. **博客展示层** — 将生成的文档通过 GitHub Pages 展示为个人博客

具体实现方式待后续确定。
