---
layout: home
title: "AI Studio"
author_profile: true
---

欢迎来到 AI Studio！

这里是我使用 AI 技术生成的学习文档和总结。通过 GitHub Pages 构建的个人博客，记录 AI 学习之路。

## 最近更新

{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
