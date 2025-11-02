---
layout: post
mathjax: true
title: "CSS 快速入门 3"
subtitle: "CSS Quick Start"
date: 2025-11-02 17:22:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - CSS
---

# CSS 快速入门

## CSS 样式设计与应用

### 超链接样式的特点

超链接样式的特殊性：  
文本或图像加上链接，将失去原有的样式，变成浏览器默认的超链接样式。

超链接样式的四种形态：
- 未访问状态(`a:link`)
- 已访问状态(`a:visited`)
- 鼠标悬停状态(`a:hover`)
- 激活选定状态(`a:active`)

未访问的链接（`a:link`）默认颜色为蓝色，已访问的链接（`a:visited`）默认颜色为紫色。  
鼠标悬停时（`a:hover`）和点击时（`a:active`）的样式也可以单独设置。  

超链接伪类形式：

<!-- 表格： -->

| 伪类选择器  | 示例                     | 说明                                       |
| ----------- | ------------------------ | ------------------------------------------ |
| `a:link`    | `a:link{color:#999;}`    | 默认状态下的链接样式                       |
| `a:visited` | `a:visited{color:#666;}` | 已访问链接的样式                           |
| `a:hover`   | `a:hover{color:#f00;}`   | 鼠标悬停状态下的链接样式                   |
| `a:active`  | `a:active{color:#0f0;}`  | 激活选定状态(鼠标点击未释放时)下的链接样式 |

示例：

```css
a:link {
    color: #0000FF; /* 未访问链接颜色 */
    text-decoration: none; /* 去掉下划线 */
}
a:visited {
    color: #800080; /* 已访问链接颜色 */
}
a:hover {
    color: #FF0000; /* 鼠标悬停颜色 */
    text-decoration: underline; /* 添加下划线 */
}
a:active {
    color: #00FF00; /* 激活选定颜色 */
}
```

### 应用样式的三种方式

在 CSS 中，应用样式有三种主要方式：内联样式、内部样式表和外部样式表。

1. **内联样式（Inline Styles）**：  
   直接在 HTML 元素的 `style` 属性中定义样式。   
   示例：
   ```html
   <a href="https://example.com" style="color: red; text-decoration: none;">内联样式链接</a>
   ```
2. **内部样式表（Internal Stylesheet）**：  
   在 HTML 文档的 `<head>` 部分使用 `<style>` 标签定义样式。  
     示例：
     ```html
     <head>
         <style>
             a {
                 color: blue;
                 text-decoration: none;
             }
         </style>
     </head>
     <body>
         <a href="https://example.com">内部样式表链接</a>
     </body>
     ```


3. **外部样式表（External Stylesheet）**：
    将样式定义在单独的 CSS 文件中，并通过 `<link>` 标签将其链接到 HTML 文档中。  
    示例：
    ```html
    <head>
        <link rel="stylesheet" href="styles.css">
    </head>
    <body>
        <a href="https://example.com">外部样式表链接</a>
    </body>
    ```
    `styles.css` 文件内容：
    ```css
    a {
        color: green;
        text-decoration: none;
    }
    ```

总结：
- 内联样式优先级最高，但不利于维护，应尽量少用或不用。
- 内部样式表适用于单个页面的样式定义。
- 外部样式表适用于多个页面共享的样式定义，便于维护和管理。

>  提示：为避免 padding 和 border 导致元素实际占用空间超过预期，推荐在项目中全局启用 `box-sizing: border-box;`：
>
>  ```css
>  *, *::before, *::after { box-sizing: border-box; }
>  ```
>
>  这会让元素的 `width`/`height` 包含内边距和边框，通常更利于布局计算和响应式设计。

### 样式优先级

在 CSS 中，当多个样式规则应用于同一元素时，浏览器会根据一定的优先级规则来决定最终应用哪一个样式。样式优先级从高到低依次为：
1. 内联样式（Inline Styles）
2. 内部样式表（Internal Stylesheet）
3. 外部样式表（External Stylesheet）
4. 浏览器默认样式

<br>

1. ID 选择器（ID Selector）
2. 类选择器（Class Selector）
3. 标签选择器（Element Selector）

> 说明：样式最终由两个主要因素决定——选择器的特指性（specificity）和来源顺序（cascade）。通常先比较特指性（从高到低：内联样式 > ID > 类/属性/伪类 > 元素/伪元素），如果特指性相同则按来源顺序决定（后声明覆盖先声明）。另外，`!important` 可以提升声明的优先级，但应慎用。

例如，假设有以下样式规则：

```css
#nav_id { width: 300px; background: #ccc; }          /* ID 选择器 */
.nav { height: 100px; background: red; }  /* 类选择器 */
div { border: 1px solid green; background-color: blue; }     /* 标签选择器；注意：使用 background-color 更精确，background 是简写会重置其它背景相关子属性 */
```

以及以下 HTML 结构：

```html
<body>
  <div id="nav_id" class="nav">
    <ul>
      <li>首页</li>
      <li>关于我们</li>
      <li>联系我们</li>
    </ul>
  </div>
</body>
```

最终，`div` 元素的样式将会是：
- 宽度：300px（来自 ID 选择器）
- 高度：100px（来自类选择器）
- 边框：1px 实线绿色（来自标签选择器）
- 背景色：#ccc（来自 ID 选择器，优先级最高）

### 复用样式

为了提高代码的复用性和维护性，建议使用类选择器（Class Selector）来定义样式。

```css
.pic1 {width: 28px; height: 26px; background-color: blue; border: 1px solid #000;}
.pic2 {width: 28px; height: 26px; background-color: yellow; border: 1px solid #000;}
.pic3 {width: 28px; height: 26px; background-color: red; border: 1px solid #000;}
```

可以复用样式，简化代码：

```css
.pic { width: 28px; height: 26px; border: 1px solid #000;}
.pic-blue { background-color: blue; }
.pic-yellow { background-color: yellow; }
.pic-red { background-color: red; }
```

```html
<div class="pic pic-blue">绿</div>
<div class="pic pic-yellow">黄</div>
<div class="pic pic-red">红</div>
```

### 组合样式

基本符号：
- 逗号（,）：表示多个选择器的组合，逗号前后选择器的样式相同。
- 空格（ ）：表示后代选择器，选择某个元素内部的指定子元素。
- #：表示ID选择器。
- 冒号（:）：表示伪类选择器。

组合示例：

标签+类：
```css
div.box { border: 1px solid #000; }
```
标签+ID：
```css
div#header { height: 100px; }
```
ID+空格+标签：
```css
#nav ul { list-style: none; }
```
伪类+标签：
```css
a:hover { color: #f00; }
```