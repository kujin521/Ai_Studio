# GitHub 个人博客实现文档

## 概述

将 Markdown 文档上传到 GitHub，然后以个人博客网页的形式展示。本仓库（Ai_Studio）中由 AI 生成的学习文档和总结，通过此方式发布为可公开访问的个人博客页面。

---

## 方案一：GitHub Pages + Jekyll（推荐 ⭐）

**特点**：GitHub 原生支持，无需额外构建工具，推送 MD 文件即可自动发布。

### 原理

GitHub Pages 内置 Jekyll 静态站点生成器，它会自动将 `username.github.io` 仓库中的 `.md` 文件渲染为 HTML 页面。

### 操作步骤

#### 1. 创建 GitHub Pages 仓库

```
方式 A：个人/组织站点（一个账号只能创建一个）
仓库名：<你的用户名>.github.io
例：zhangsan.github.io

方式 B：项目站点（每个仓库都可以创建一个）
在 Ai_Studio 仓库中启用 GitHub Pages 即可
```

#### 2. 安装 Jekyll（本地预览，可选）

```bash
# 安装 Ruby + Bundler（Windows 用 RubyInstaller）
gem install jekyll bundler
```

#### 3. 初始化 Jekyll 站点

```bash
# 在 Ai_Studio 目录下执行
jekyll new . --force
```

#### 4. 目录结构

```
Ai_Studio/
├── _config.yml          # Jekyll 配置文件
├── _posts/              # 博客文章（Markdown）
│   └── 2026-06-19-AI学习总结.md
├── _layouts/            # 页面模板
├── index.md             # 首页
├── about.md             # 关于页
└── assets/              # 静态资源（CSS/JS/图片）
```

#### 5. 写一篇文章

在 `_posts/` 目录创建文件，命名格式必须为：`年-月-日-标题.md`

```markdown
---
layout: post
title: "AI 学习总结"
date: 2026-06-19
categories: AI
---

这是一篇由 AI 生成的学习文档……
```

#### 6. 发布到 GitHub Pages

```bash
# 方式 A：个人站点
git remote add origin https://github.com/<用户名>/<用户名>.github.io.git
git push -u origin master
# 访问 https://<用户名>.github.io

# 方式 B：项目站点
git push origin master
# 到仓库 Settings → Pages → 选择 master 分支 /docs 文件夹
# 访问 https://<用户名>.github.io/Ai_Studio/
```

#### 7. 选择主题

Jekyll 支持大量主题，推荐：

| 主题 | 风格 | GitHub Stars |
|------|------|-------------|
| [Minimal Mistakes](https://github.com/mmistakes/minimal-mistakes) | 功能丰富，适合博客 | 12k+ |
| [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) | 简洁美观，适合技术博客 | 7k+ |
| [TeXt](https://github.com/kitian616/jekyll-TeXt-theme) | 文档型，适合学习笔记 | 2k+ |

安装主题：在 `_config.yml` 中设置 `theme: minimal-mistakes-jekyll`

---

## 方案二：GitHub Pages + VitePress（推荐 ⭐）

**特点**：基于 Vue 3 + Vite，构建快速，支持 Vue 组件扩展，适合技术文档站。

### 操作步骤

#### 1. 初始化项目

```bash
# 在 Ai_Studio 目录下
npm init -y
npm install -D vitepress
```

#### 2. 创建目录结构

```
Ai_Studio/
├── docs/
│   ├── .vitepress/
│   │   └── config.js      # VitePress 配置
│   ├── index.md           # 首页
│   ├── ai-notes/          # AI 学习笔记
│   │   └── index.md
│   └── public/            # 静态资源
├── package.json
└── .github/
    └── workflows/
        └── deploy.yml     # GitHub Actions 部署配置
```

#### 3. 配置 VitePress

`docs/.vitepress/config.js`：

```javascript
export default {
  title: 'AI Studio 学习笔记',
  description: 'AI 技术学习文档与总结',
  themeConfig: {
    nav: [
      { text: '首页', link: '/' },
      { text: 'AI 笔记', link: '/ai-notes/' },
    ],
    sidebar: [
      { text: 'AI 学习总结', link: '/ai-notes/' },
    ]
  }
}
```

#### 4. 配置 GitHub Actions 自动部署

`.github/workflows/deploy.yml`：

```yaml
name: Deploy VitePress site

on:
  push:
    branches: [master]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install
      - run: npm run docs:build
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: docs/.vitepress/dist
```

`package.json` 添加脚本：

```json
{
  "scripts": {
    "docs:dev": "vitepress dev docs",
    "docs:build": "vitepress build docs",
    "docs:preview": "vitepress preview docs"
  }
}
```

#### 5. 发布

```bash
git push origin master
# 仓库 Settings → Pages → 选择 gh-pages 分支
# 访问 https://<用户名>.github.io/Ai_Studio/
```

---

## 方案三：GitHub Pages + Hugo

**特点**：Go 语言编写，构建速度极快，主题丰富，适合内容较多的博客。

### 操作步骤

#### 1. 安装 Hugo

```bash
# Windows：从 https://gohugo.io/installation/ 下载
# 或用 Chocolatey
choco install hugo-extended
```

#### 2. 创建站点

```bash
hugo new site Ai_Studio --force
cd Ai_Studio
git init
```

#### 3. 添加主题

```bash
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke
echo "theme = 'ananke'" >> hugo.toml
```

#### 4. 创建文章

```bash
hugo new content posts/AI学习总结.md
```

#### 5. 配置 GitHub Actions

`.github/workflows/hugo-deploy.yml`：

```yaml
name: Deploy Hugo site

on:
  push:
    branches: [master]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true
      - uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
      - run: hugo --minify
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

---

## 方案对比

| 方案 | 学习成本 | 灵活性 | 构建速度 | GitHub 原生支持 | 适用场景 |
|------|---------|--------|---------|----------------|---------|
| **Jekyll** | ⭐⭐⭐ 中 | 中 | 慢 | ✅ 完全支持 | 纯博客、文章站 |
| **VitePress** | ⭐⭐⭐⭐ 低 | 高 | 快 | ❌ 需 Actions | 文档站、技术笔记 |
| **Hugo** | ⭐⭐ 中高 | 高 | 极快 | ❌ 需 Actions | 大型内容站 |

**推荐选择**：
- 如果你只想写 Markdown 不想折腾 → **Jekyll**（零配置）
- 如果你熟悉前端技术栈 → **VitePress**（现代化开发体验）
- 如果你内容量巨大（100+ 篇文章） → **Hugo**（构建速度最快）

---

## 本项目推荐方案

Ai_Studio 项目当前状态为空项目，建议采用 **Jekyll（方案一）**：

### 理由

1. **零构建配置** — GitHub Pages 原生支持 Jekyll，提交即可用
2. **聚焦内容** — 项目核心是使用 AI 生成学习文档，不应在搭建博客上花过多精力
3. **MD 友好** — 在 `_posts/` 目录写 MD 文件即可自动发布，与 AI 生成工作流无缝衔接
4. **低成本入门** — 不需要额外学习前端框架就能上线

### 建议步骤

```
1. 创建 <用户名>.github.io 仓库
2. 用 Jekyll + Minimal Mistakes 主题搭建
3. 每次 AI 生成学习总结后，放入 _posts/ 目录
4. git push 自动发布
5. 后续可切换到 VitePress 获得更好的定制体验
```

---

## 写作规范（通用）

所有发布的 Markdown 文档建议遵循以下规范，以保证在任意博客框架下都能正确渲染：

```markdown
---
# 文章元信息（Front Matter）— Jekyll/VitePress 均支持
title: 文章标题
date: 2026-06-19
tags: [AI, 深度学习]
description: 文章简介，会显示在搜索结果和分享卡片中
---

## 正文使用标准 Markdown 语法

### 代码块

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello AI Studio");
    }
}
```

### 图片

![替代文字](assets/images/example.png)

### 链接

[外部链接](https://example.com)
[内部页面](/ai-notes/summary/)
```
