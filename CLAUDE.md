# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概况

- **项目名称**: Ai_Studio
- **目的**: 学习 AI 技术，利用 AI 生成学习文档和总结，并通过 GitHub 构建个人博客页面
- **当前状态**: 博客已搭建完成，已有 4 篇 AI 生成的学习文档
- **IDE**: IntelliJ IDEA
- **版本控制**: Git（master 分支，远程: https://github.com/kujin521/Ai_Studio.git）
- **博客地址**: https://kujin521.github.io/Ai_Studio/

## 项目结构

```
Ai_Studio/
├── _config.yml                           # Jekyll 配置（Minimal Mistakes 主题）
├── Gemfile                               # Ruby 依赖管理
├── index.md                              # 博客首页
├── about.md                              # 关于页
├── 404.html                              # 自定义 404 页面
├── .gitignore                            # 排除 IDE/构建文件
├── CLAUDE.md                             # 本文件
├── 项目说明.md                            # 原始项目描述
├── GitHub个人博客实现文档.md              # 博客搭建方案对比文档
├── README.md                             # GitHub 仓库首页说明
├── assets/images/                        # 图片资源目录
├── _posts/                               # 博客文章目录（.md 文件）
│   ├── 2026-06-19-AI-Studio-项目启动.md
│   ├── 2026-06-19-Claude-Code-简介与入门.md
│   ├── 2026-06-19-Claude-Code-核心功能详解.md
│   └── 2026-06-19-Claude-Code-实战技巧与最佳实践.md
└── _pages/                               # 自定义页面目录
```

## 技术栈

- **博客框架**: Jekyll（GitHub Pages 原生渲染）
- **主题**: Minimal Mistakes（`remote_theme: mmistakes/minimal-mistakes`）
- **构建**: GitHub Actions 自动构建，push 即发布
- **语言**: 中文（`locale: zh-CN`）

## 开发命令

```bash
# 本地预览博客（需安装 Ruby + Jekyll）
bundle exec jekyll serve

# 新增文章
# 在 _posts/ 目录创建 YYYY-MM-DD-标题.md，必须包含 Front Matter:
# ---
# title: "文章标题"
# date: 2026-06-19
# categories: [AI]
# tags: [AI, 学习]
# ---

# 提交发布
git add _posts/
git commit -m "add: 新文章标题"
git push origin master
# → GitHub Pages 自动构建并发布
```

## 博客文章规范

- 文件名: `YYYY-MM-DD-标题.md`（必须）
- 每篇文章开头必须有 Jekyll Front Matter（title/date/categories/tags）
- 文章放入 `_posts/` 目录（放在其他位置不会被渲染）
- 提交到 master 分支后，GitHub Pages 自动构建，约 1-2 分钟生效

## Git 注意事项

- Windows 环境下需配置 `safe.directory` 才能正常使用 git 命令
- 代理命令: `git -c safe.directory=E:/Code/Demo/Ai_Studio <command>`
- 提交时需指定 user 信息: `-c user.email=... -c user.name=...`

## 架构说明

项目分为两层：

1. **AI 学习文档生成层** — 利用 Claude Code 生成技术学习文档和总结，产出 Markdown 文件
2. **博客展示层** — 通过 GitHub Pages + Jekyll 将生成的文档自动发布为个人博客

工作流: 用 Claude Code 生成 MD 文档 → 放入 `_posts/` → `git push` → 自动上线
