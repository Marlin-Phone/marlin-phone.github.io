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
---

# HTML 快速入门

## HTML 基础结构与标签

### HTML 标记

标记类型分为单标记和双标记：  
- 单标记例如 `<br>`  
- 双标记例如 `<p></p>`  

标记属性分为一般属性和事件属性：
- 一般属性例如 `<标记 属性1="属性值" 属性2="属性值">`
  示例：`<hr size="3" align="center" width="50">`

事件属性的值通常是一个 JavaScript 函数。
示例：`<body onload="check()">`

### 文件头及功能

`<head>...</head>` 标记对之间的部分称为文件头。

### 子标记

`<title></title>`: 网页摘要信息，利于浏览器解析和搜索引擎搜索。

`<meta>`: 单标记，提供两类元数据描述：  
- 关于 HTTP 头的描述，描述文档类型和字符编码向浏览器返回信息，以正确显示网页内容: `<meta http-equiv="Content-Type" content="text/html; charset=gb2312">`  
- 关于页面信息的描述，用于搜索引擎: `<meta name="keywords" content="淘宝，网上购物在线交易，交易市场">`

`<base>`
`<link>`
`<style></style>`

#### 标题标签

`<h1>一级标题</h1>`
`<h2>二级标题</h2>`
`<h3>三级标题</h3>`
`<h4>四级标题</h4>`
`<h5>五级标题</h5>`
`<h6>六级标题</h6>`

#### 段落标签

`<p>第一段</p>`
`<p>第二段</p>`

#### 水平线标签

`<hr>`
`<hr />` - 单个标签的闭合形式（XHTML 写法）  
两者效果一样，写法不同。  
所有的单标记都有 `<标记>` 或 `<标记 />` 这两种写法。  
示例：
`<hr>`

#### 有序列表标签

`<ol>` 全称为 Ordered List (有序列表)  
`<li>` 全称为 List Item (列表项)，必须放在 `<ol>` 或 `<ul>` 内。  
示例：

```html
<ol>
  <li>列表项1</li>
  <li>列表项2</li>
</ol>
```

<ol>
  <li>列表项1</li>
  <li>列表项2</li>
</ol>

#### 无序列表标签

`<ul>` 全称为 Unordered List (无序列表)  
示例：

```html
<ul>
  <li>列表项1</li>
  <li>列表项2</li>
</ul>
```

<ul>
  <li>列表项1</li>
  <li>列表项1</li>
</ul>

#### 定义描述标签

`<dl>` 全称为 Description List (定义列表)  
`<dt>` 全称为 Definition Term (定义术语/条目)  
`<dd>` 全称为 Definition Description (定义描述)  
示例：

```html
<dl>
  <dt>咖啡</dt>
  <dd>一种黑色的热饮料，主要由烘焙咖啡豆制成。</dd>
</dl>
```
<dl>
  <dt>咖啡</dt>
  <dd>一种黑色的热饮料，主要由烘焙咖啡豆制成。</dd>
</dl>


#### 表格标签

`<tr>` 全称为 Table Row (表格行)  
`<td>` 全称为 Table Data (表格数据/单元格)  

```html
<table>
  <tr>
    <td>百度</td>
    <td>新浪</td>
  </tr>
  <tr>
    <td>虎扑</td>
    <td>谷歌</td>
  </tr>
</table>
```

<table>
  <tr>
    <td>百度</td>
    <td>新浪</td>
  </tr>
  <tr>
    <td>虎扑</td>
    <td>谷歌</td>
  </tr>
</table>

#### 分区标签

`<div>` 全称为 division (分区)

```html
<body>
  <div>
    <h3>标题</h3>
    <p>内容</p>
  </div>
</body>
```

#### 表单

```html
<form>
  <table>
    <tr>
      <td>...</td>
      <td>...</td>
    </tr>
  </table>
</form>
```

### 实际开发中常用的四种块状结构

1. `div-ul(ol)-li`: 常用于分类导航或菜单等
2. `div-dl-dt-dd`: 常用于图文混编的场合
3. `table-tr-td`: 常用于图文布局或显示数据
4. `form-table-tr-td`: 常用于布局表单

#### 图像标签

`<img>` 全称为 image  
`src` 表示资源地址（source）  
`alt` 表示替代文本（alternative text），用于描述图片内容，供屏幕阅读器和图片无法显示时使用  
`title` 通常在鼠标悬停时显示

```html
<img src="图片地址" alt="提示文字" title="提示文字">
```

eg:

```html
<img src="img\星空.jpg" alt="图片" title="一张图片">
```

<img src="img\星空.jpg" alt="图片" title="一张图片">

#### 范围标签

示例：

```html
<p>
  商品价格，仅售
  <span style="color:red;font-size:70px;">10</span>
  元
</p>
```

<p>商品价格，仅售
<span style="color:red;font-size:70px;">10</span>
元
</p>

#### 换行标签

示例：

```html
第一行<br>第二行
```

第一行<br>第二行

## XHTML 1.0 基本规范

- 标签名和属性名称必须小写
- HTML 标签必须关闭
- 属性值必须用引号括起来
- 标签必须正确嵌套
- 必须添加文档类型声明
