---
layout: post
mathjax: true
title: "CSS 快速入门 2"
subtitle: "CSS Quick Start"
date: 2025-11-02 16:01:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - CSS
---

# CSS 快速入门

## CSS 盒模型与应用

### CSS 盒模型

CSS 盒模型（Box Model）是网页布局的基础概念，它描述了每个 HTML 元素在页面上所占据的空间。每个元素都被看作一个矩形盒子，盒子由以下四个部分组成：

- **内容区域（Content Area）**：显示文本和图像的区域。
- **内边距（Padding）**：内容区域与边框之间的空间，透明。
- **边框（Border）**：围绕内容和内边距的边框区域。
- **外边距（Margin）**：边框与其他元素之间的空间，透明。

```css
/* 示例 */
.box {
    width: 300px;          /* 内容区域宽度 */
    padding: 20px;        /* 内边距 */
    border: 5px solid #000; /* 边框 */
    margin: 30px;         /* 外边距 */
}
```

```html
<div class="box">内容</div>
```

上述代码定义了一个类名为 `box` 的盒子，其内容区域宽度为 300 像素，内边距为 20 像素，边框宽度为 5 像素，外边距为 30 像素。

除了内容外，各盒属性又可细分为四个方向的属性：  
`top`、`right`、`bottom`、`left`，分别对应上、右、下、左四个方向（顺时针默认）：

- 内边距：`padding-top`、`padding-right`、`padding-bottom`、`padding-left`
- 边框：`border-top`、`border-right`、`border-bottom`、`border-left`
- 外边距：`margin-top`、`margin-right`、`margin-bottom`、`margin-left`

- **内容**：`width`、`height`

各方向属性可以单独设置，也可以使用简写形式（顺时针，从上开始）：

```css
.box {
  /* 四个值：上 右 下 左 */
  padding: 10px 20px 15px 25px; /* 上 右 下 左 */

  /* 三个值：上 水平(左右) 下 */
  /* padding: 上 右/左 下 */
  padding: 10px 20px 30px; /* 上：10px；左右：20px；下：30px */

  /* 两个值：垂直(水平) */
  margin: 5px 10px; /* 上下：5px；左右：10px */

  /* 一个值：四边相同 */
  border-width: 2px; /* 四个方向边框宽度相同 */
}
```

### 盒模型的计算

盒模型的总宽度和高度计算公式如下：
- 总宽度 = 内容宽度 + 左右内边距 + 左右边框 + 左右外边距
- 总高度 = 内容高度 + 上下内边距 + 上下边框 + 上下外边距

