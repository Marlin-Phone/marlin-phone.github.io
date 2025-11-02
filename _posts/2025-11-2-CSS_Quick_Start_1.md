---
layout: post
mathjax: true
title: "CSS 快速入门 1"
subtitle: "CSS Quick Start"
date: 2025-11-02 15:39:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - CSS
---

# CSS 快速入门

## CSS 基础语法与选择器

### CSS 基础语法

CSS（Cascading Style Sheets，层叠样式表）用于描述 HTML 文档的外观和格式。CSS 语法由选择器和声明块组成。

```html
<head>
    <style>
        选择器 {
            对象的属性1: 属性值1;
            对象的属性2: 属性值2;
        }
    </style>
</head>
```

style 标签用于在 HTML 文档中嵌入 CSS 样式。

示例：

```css
li{
    color: blue;
    font-size: 20px;
    font-family: 隶书;
}
```

上述代码将所有 `<li>` 元素的文本颜色设置为蓝色，字体大小为 20 像素，字体为隶书。

### CSS 选择器

CSS 选择器用于选择要应用样式的 HTML 元素。常见的选择器有：

- **元素选择器**：选择所有指定类型的元素。例如，`li` 选择所有 `<li>` 元素。
- **类选择器**：选择所有具有指定类的元素。例如，`.classname` 选择所有类名为 `classname` 的元素。
- **ID 选择器**：选择具有指定 ID 的元素。例如，`#idname` 选择 ID 为 `idname` 的元素。
- **属性选择器**：选择具有指定属性的元素。例如，`[type="text"]` 选择所有 `type` 属性为 `text` 的元素。

```css
/* 元素选择器 */
li {
    color: blue;
}

/* 类选择器 */
.highlight {
    background-color: yellow;
}

/* ID 选择器 */
#main-title {
    font-size: 24px;
}

/* 属性选择器 */
input[type="text"] {
    border: 1px solid #ccc;
}
```

### 文本属性

CSS 提供了多种文本属性来控制文本的外观和布局。常用的文本属性包括：  
字体、字号：  
- `font`：简写属性，用于设置字体样式、大小和字体族。
- `font-weight`：设置字体粗细（如正常、加粗等）。
- `font-size`：设置文本大小。
- `font-family`：设置文本字体。

行距、对齐：
- `line-height`：设置行间距。
- `text-align`：设置文本对齐方式（如左对齐、右对齐、居中）。
- `letter-spacing`：设置字母间距。
- `text-decoration`：设置文本装饰（如下划线、删除线等）。
- `white-space`：控制空白符的处理方式。

示例：

```css
h1 {
    font: bold 24px "隶书";
    line-height: 1.5;
    text-align: center;
    letter-spacing: 2px;
    text-decoration: underline;
    white-space: nowrap;
}
```

上述代码将所有 `<h1>` 元素的字体设置为加粗的 24 像素隶书，行间距为 1.5 倍，文本居中对齐，字母间距为 2 像素，并添加下划线装饰，同时禁止换行。

### 背景属性

- `background`：简写属性，用于设置背景颜色、图像等。
- `background-color`：设置背景颜色。
- `background-image`：设置背景图像。
- `background-repeat`：设置背景图像的重复方式。
- `background-position`：设置背景图像的位置。

示例：

```css
body {
    background-color: #f0f0f0;
    background-image: url("img/background.jpg");
    background-repeat: no-repeat;
    background-position: center center;
    /* 以上内容等价于
    background: #f0f0f0 url("img/background.jpg") no-repeat center center;
    */
}
```