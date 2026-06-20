# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概况

- **项目名称**: Ai_Studio
- **目的**: 学习 AI 技术，利用 AI 生成学习文档和总结，并通过 GitHub 构建个人博客页面
- **当前状态**: 博客已迁移为 Chirpy 主题，已有 Claude Code 基础系列与 Spring Boot 深度实战系列文档
- **IDE**: IntelliJ IDEA
- **版本控制**: Git（master 分支，远程: https://github.com/kujin521/Ai_Studio.git）
- **博客地址**: https://kujin521.github.io/Ai_Studio/

## 项目结构

```
Ai_Studio/
├── _config.yml                           # Jekyll 配置（Chirpy 主题）
├── Gemfile                               # Ruby 依赖管理
├── index.html                            # Chirpy 首页入口
├── .github/workflows/pages-deploy.yml    # GitHub Pages Actions 部署流程
├── .gitignore                            # 排除 IDE/构建文件
├── CLAUDE.md                             # 本文件
├── 项目说明.md                            # 原始项目描述
├── GitHub个人博客实现文档.md              # 博客搭建方案对比文档
├── README.md                             # GitHub 仓库首页说明
├── assets/                               # Chirpy 静态资源
├── _data/                                # Chirpy 数据与本地化配置
├── _sass/                                # Chirpy 样式源码
├── _tabs/                                # Chirpy 左侧导航页面
│   ├── about.md
│   ├── archives.md
│   ├── categories.md
│   ├── tags.md
│   └── spring-boot-series.md             # Spring Boot 系列目录页
└── _posts/                               # 博客文章目录（.md 文件）
    ├── 2026-06-19-*.md                   # 已发布旧文章
    └── 2026-06-20-*.md                   # Spring Boot 深度实战系列文章
```

## 技术栈

- **博客框架**: Jekyll（GitHub Pages 原生渲染）
- **主题**: Chirpy（`theme: jekyll-theme-chirpy`）
- **构建**: GitHub Actions 自动构建，push 到 master 后发布
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
- 同一系列文章必须使用统一文件名前缀，例如：`2026-06-20-Claude-Code-Spring-Boot-深度实战-01-概述.md`
- 同一系列文章标题必须带序号，保证 Chirpy 归档和搜索中可读
- 提交到 master 分支后，GitHub Pages 自动构建，约 1-2 分钟生效

## 系列目录与导航规范

生成多篇系列文档时，必须同步维护目录导航：

1. 在 `_tabs/` 下创建或更新系列目录页，例如 `_tabs/spring-boot-series.md`。
2. 系列目录页必须包含 Front Matter：`title`、`icon`、`order`、`permalink`。
3. 系列目录页必须列出该系列所有文章，链接使用 Jekyll `post_url`，例如：
   ```markdown
   [概述与学习路线]({% post_url 2026-06-20-Claude-Code-Spring-Boot-深度实战-01-概述 %})
   ```
4. 每篇系列文章末尾必须追加 `## 系列导航`，至少包含：
   - 返回系列目录：`[返回系列目录]({{ '/spring-boot-series/' | relative_url }})`
   - 上一篇（第一篇除外）
   - 下一篇（最后一篇除外）
5. 新增、删除或重命名系列文章时，必须同步更新：
   - `_tabs/<series>.md` 系列目录页
   - 相邻文章的上一篇/下一篇链接
   - 当前文章末尾的系列导航
6. 不要只依赖 Chirpy 默认 Archives/Categories/Tags；它们只能按时间、分类、标签浏览，不能替代系列阅读目录。

## Chirpy 导航说明

- Chirpy 左侧栏显示 `_tabs/` 中的页面，不会自动显示所有 `_posts/` 文章。
- 如果希望左侧出现某个文档集合，应该在 `_tabs/` 中创建一个系列目录页。
- 如需“左侧直接展开全部文章”，需要定制 Chirpy 主题布局，维护成本较高，默认不建议。

## Git 注意事项

- Windows 环境下需配置 `safe.directory` 才能正常使用 git 命令
- 代理命令: `git -c safe.directory=E:/Code/Demo/Ai_Studio <command>`
- 提交时需指定 user 信息: `-c user.email=... -c user.name=...`

## 架构说明

项目分为两层：

1. **AI 学习文档生成层** — 利用 Claude Code 生成技术学习文档和总结，产出 Markdown 文件
2. **博客展示层** — 通过 GitHub Pages + Jekyll 将生成的文档自动发布为个人博客

工作流: 用 Claude Code 生成 MD 文档 → 放入 `_posts/` → `git push` → 自动上线
