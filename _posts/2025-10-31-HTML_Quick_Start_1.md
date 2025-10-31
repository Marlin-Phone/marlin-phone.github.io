---
layout: post
mathjax: true
title: "HTML 快速入门 1"
subtitle: "HTML Quick Start"
date: 2025-10-31 20:33:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - HTML
  - CSS
---

# HTML 快速入门

## HTML 基础结构与标签

### HTML 文档结构

HTML 文档由以下部分组成：
- `<!DOCTYPE>`：文档类型声明，指定 HTML 版本。
- `<html>`：根元素，包含整个文档。
- `<head>`：文档头部，包含元信息。
- `<body>`：文档主体，包含可见内容。

示例：

```html
<!DOCTYPE html>
<html>
<head>
  <title>示例页面</title>
</head>
<body>
  <h1>欢迎</h1>
</body>
</html>
```

### 文件头及功能

`<head>` 中包含：
- `<title>`：网页标题。
- `<meta>`：元信息（如字符编码）。
- `<link>`：外部资源链接（如 CSS）。
- `<style>`：内联样式。

示例：

```html
<head>
  <meta charset="utf-8">
  <title>示例页面</title>
  <link rel="stylesheet" href="styles.css">
</head>
```

## 常用 HTML 标签

### 标题与段落

- 标题：`<h1>` ~ `<h6>`
- 段落：`<p>`

示例：

```html
<h1>一级标题</h1>
<p>这是一个段落。</p>
```

### 列表

- 有序列表：`<ol>`
- 无序列表：`<ul>`
- 列表项：`<li>`

示例：

```html
<ul>
  <li>无序项1</li>
  <li>无序项2</li>
</ul>

<ol>
  <li>有序项1</li>
  <li>有序项2</li>
</ol>
```

### 表格

- 表格行：`<tr>`
- 表格单元格：`<td>`

示例：

```html
<table>
  <tr>
    <td>名称</td>
    <td>价格</td>
  </tr>
  <tr>
    <td>商品A</td>
    <td>100元</td>
  </tr>
</table>
```

### 图像

- `src`：图片路径。
- `alt`：替代文本。
- `title`：提示信息。

示例：

```html
<img src="img/starry-sky.jpg" alt="星空" title="夜晚的星空">
```

### 表单

表单用于用户输入：

```html
<form action="/submit" method="post">
  <label for="name">姓名：</label>
  <input id="name" name="name" type="text">
  <button type="submit">提交</button>
</form>
```

## 开发实践

### 块状结构

1. `div-ul-li`：导航菜单。
2. `div-dl-dt-dd`：图文混排。
3. `table-tr-td`：数据表格。
4. `form-table-tr-td`：表单布局。

### XHTML 与 HTML5

- XHTML：严格语法，标签必须闭合。
- HTML5：宽松语法，推荐使用。

示例（HTML5）：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>HTML5 示例</title>
</head>
<body>
  <h1>欢迎</h1>
</body>
</html>
```
