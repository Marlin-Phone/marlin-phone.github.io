# 我的博客仓库

<p align="center">
  <a href="https://wakatime.com/badge/user/72f7b5ae-3c4b-48e8-a41a-2f941eeb7e9d/project/ddcc7af3-5ba2-4553-8e36-532f3158646e"><img src="https://wakatime.com/badge/user/72f7b5ae-3c4b-48e8-a41a-2f941eeb7e9d/project/ddcc7af3-5ba2-4553-8e36-532f3158646e.svg" alt="wakatime">
  </a>
  <img src="https://img.shields.io/github/last-commit/marlin-phone/Marlin-Phone.github.io?logo=github&color=success" alt="last commit"/>
  <img src="https://img.shields.io/github/commit-activity/w/marlin-phone/Marlin-Phone.github.io" alt="commit activity"/>
  <img src="https://visitor-badge.laobi.icu/badge?page_id=marlin-phone.Marlin-Phone.github.io" alt="visitors"/>
  <img src="https://img.shields.io/github/languages/top/marlin-phone/Marlin-Phone.github.io?logo=c%2B%2B&logoColor=white" alt="top language"/>
  <img src="https://img.shields.io/github/license/marlin-phone/Marlin-Phone.github.io" alt="license"/>
</p>

## 项目介绍

这是我的个人博客仓库，基于 [Hux Blog](https://github.com/Huxpro/huxpro.github.io) 模板构建，使用 Jekyll 静态站点生成器。博客内容主要包含：

- 算法学习笔记和刷题心得
- 计算机专业知识总结（数据结构、算法、网络等）
- 竞赛经历和心得（蓝桥杯、数学建模等）
- 技术教程和开发经验分享

博客已部署在 GitHub Pages，访问地址：[https://marlin-phone.github.io/](https://marlin-phone.github.io/)

## 技术栈

- **静态站点生成器**: [Jekyll](https://jekyllrb.com/)
- **前端框架**: Bootstrap 3
- **部署平台**: GitHub Pages
- **构建工具**: Grunt
- **语言支持**: Markdown, HTML, CSS, JavaScript

## 安装与运行

### 环境要求

- Ruby 2.7+
- Bundler
- Node.js (可选，用于Grunt任务)

### 本地运行步骤

1. 克隆仓库:
```bash
git clone https://github.com/Marlin-Phone/Marlin-Phone.github.io.git
cd Marlin-Phone.github.io
```

2. 安装依赖:
```bash
# 安装 Ruby 依赖
bundle install

# 安装 Node.js 依赖（可选）
npm install
```

3. 启动本地服务:
```bash
# 仅启动 Jekyll 服务
bundle exec jekyll serve

# 同时启动 Grunt 监听（自动编译 CSS/JS）
npm run dev
```

4. 在浏览器中访问:
```
http://localhost:4000
```

## 主要功能特性

### 1. 响应式设计
- 适配桌面、平板和手机设备
- 移动端友好的导航菜单

### 2. 多语言支持
- 中英文双语内容支持
- 通过 URL 参数切换语言（?lang=zh 或 ?lang=en）

### 3. 内容组织系统
- **标签系统**: 通过标签分类文章内容
- **归档页面**: 按时间线组织所有文章
- **侧边栏导航**: 文章内标题导航（自动提取 H1-H6）

### 4. 搜索功能
- 内置全文搜索（基于 Simple-Jekyll-Search）
- 支持模糊匹配

### 5. 内容增强
- **代码高亮**: 使用 Rouge 语法高亮
- **数学公式**: 通过 MathJax 支持 LaTeX 公式
- **图片处理**: 响应式图片和阴影效果

### 6. 社交媒体集成
- 支持 GitHub、Bilibili、Telegram 等平台链接
- 文章分享功能

### 7. 分析与统计
- 百度统计集成
- Wakatime 代码时间追踪

### 8. PWA 支持
- 渐进式 Web 应用支持
- 离线访问能力

## 自定义配置

主要配置在 `_config.yml` 文件中，关键配置项包括：

```yaml
# 网站基本信息
title: Marlin's Blog
SEOTitle: Marlin的博客 | Marlin's Blog
description: "这里是 @Marlin 的个人博客，与你一起发现更大的世界 | 本博客正在建设中，内容不全还请见谅"
url: "https://marlin-phone.github.io"

# 头像设置
sidebar-avatar: https://raw.githubusercontent.com/Marlin-Phone/Marlin-Phone.github.io/0689be61d135cfcd5959e9726afb5111b0e69764/img/profile%20picture.jpg

# 社交媒体链接
github_username: Marlin-Phone
bilibili_username: 160313260
telegram_username: Marlin_Phone

# 评论系统（可选）
disqus_username: # 填写你的 disqus 账号

# 分析工具
ba_track_id: 0babd084cc099f7bb72cb9bce081d978 # 百度统计ID

# 侧边栏设置
sidebar: true
sidebar-about-description: "Marlin's Blog | 正在建设中······"

# 标签系统
featured-tags: true
featured-condition-size: 0
```

## 贡献指南

### 添加新文章

1. 在 `_posts` 目录下创建新文件，命名格式: `YYYY-MM-DD-文章标题.md`
2. 添加必要的 YAML 头信息:

```yaml
---
layout: post
title: "文章标题"
subtitle: "文章副标题"
date: YYYY-MM-DD HH:MM:SS
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - 标签1
  - 标签2
---
```

3. 在正文中编写 Markdown 内容

### 修改主题

1. 修改 CSS: 编辑 `less/hux-blog.less` 文件，然后运行 `grunt` 编译
2. 修改布局: 编辑 `_layouts` 目录下的相应布局文件
3. 修改包含文件: 编辑 `_includes` 目录下的相应文件

### 提交更改

1. 本地测试确认无误
2. 提交 Pull Request 到主仓库
3. 等待审核合并

## 目录结构

```
Marlin-Phone.github.io/
├── _config.yml               # Jekyll 配置文件
├── _doc/                     # 文档
├── _includes/                # 页面包含组件
├── _layouts/                 # 页面布局
├── _posts/                   # 博客文章
├── _site/                    # 生成的静态站点（自动生成）
├── css/                      # CSS 样式文件
├── img/                      # 图片资源
├── js/                       # JavaScript 文件
├── less/                     # LESS 源文件
├── pwa/                      # PWA 相关文件
├── Gemfile                   # Ruby 依赖
├── Gruntfile.js              # Grunt 构建配置
├── package.json              # Node.js 依赖
└── README.md                 # 项目说明
```

## 许可证

本项目采用 [Apache 2.0 许可证](LICENSE)。  

## 鸣谢

本博客基于 [Hux Blog](https://github.com/Huxpro/huxpro.github.io) 模板构建，感谢 Huxpro 提供的优秀模板。