---
layout: post
mathjax: true
title: "HTML 快速入门 3"
subtitle: "HTML Quick Start"
date: 2025-11-01 21:25:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - HTML
---

# HTML 快速入门

## HTML 表格及框架的应用

### 表格

语法：

```html
<table>
  <tr>
    <td>1行1列的单元格</td>
    <td>1行2列的单元格</td>
  </tr>
  <tr>
    <td>2行1列的单元格</td>
    <td>2行2列的单元格</td>
  </tr>
</table>
```

<table>
  <tr>
    <td>1行1列的单元格</td>
    <td>1行2列的单元格</td>
  </tr>
  <tr>
    <td>2行1列的单元格</td>
    <td>2行2列的单元格</td>
  </tr>
</table>

colspan 和 rowspan 属性：  
colspan 用于设置单元格跨越的列数，rowspan 用于设置单元格跨越的行数。  

border 属性：  
用于设置表格边框的宽度。

```html
<table border="1">
  <tr>
    <td colspan="2">跨两列的单元格</td>
  </tr>
  <tr>
    <td rowspan="2">跨两行的单元格</td>
    <td>普通单元格</td>
  </tr>
  <tr>
    <td>普通单元格</td>
  </tr>
</table>
```

<table border="1">
  <tr>
    <td colspan="2">跨两列的单元格</td>
  </tr>
  <tr>
    <td rowspan="2">跨两行的单元格</td>
    <td>普通单元格</td>
  </tr>
  <tr>
    <td>普通单元格</td>
  </tr>
</table>

caption、thead、tbody 和 tfoot 元素：  
`caption` 用于定义表格标题，`thead`、`tbody` 和 `tfoot` 分别用于定义表格的头部、主体和脚部。

style 属性：  
用于设置表格样式，如背景颜色等。

年终数据报表：
```html

<table width="100%">
<caption>2025年终数据报表</caption>
<thead style="background-color: #08f">      <!-- 报表的页眉 -->
    <tr>
        <th>月份</th>
        <th>销售额(万元)</th>
        <th>利润(万元)</th>
    </tr>
</thead>
<tbody>                                       <!-- 报表的主体 -->
    <tr>
        <td>一月</td>
        <td>120</td>
        <td>30</td>
    </tr>
    <tr>
        <td>二月</td>
        <td>150</td>
        <td>45</td>
    </tr>
    <tr>
        <td>三月</td>
        <td>170</td>
        <td>50</td>
    </tr>
</tbody>
<tfoot style="background-color: #00f">     <!-- 报表的页脚 -->
    <tr>
        <td>合计</td>
        <td>440</td>
        <td>125</td>
    </tr>
</tfoot>
</table>
```

<table width="100%">
<caption>2025年终数据报表</caption>
<thead style="background-color: #08f">
    <tr>
        <th>月份</th>
        <th>销售额(万元)</th>
        <th>利润(万元)</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>一月</td>
        <td>120</td>
        <td>30</td>
    </tr>
    <tr>
        <td>二月</td>
        <td>150</td>
        <td>45</td>
    </tr>
    <tr>
        <td>三月</td>
        <td>170</td>
        <td>50</td>
    </tr>
</tbody>
<tfoot style="background-color: #00f">
    <tr>
        <td>合计</td>
        <td>440</td>
        <td>125</td>
    </tr>
</tfoot>
</table>

### 框架(Frames)

框架用于在同一个浏览器窗口中显示多个HTML页面。  
框架与body标签互斥，使用`<frameset>`标签来定义框架集，使用`<frame>`标签来定义每个框架。


cols 和 rows 属性：  
用于定义框架的列数和行数。  
其值可以是具体的像素值(px)、百分比(%)，也可以是星号(*)，表示剩余空间。

```html
<frameset rows="25%,50%,*" border="1" border-color="red"> <!-- border-color属性设置边框颜色 -->
    <frame src="subframe/the_first.html">
    <frame src="subframe/the_second.html">
    <frame src="subframe/the_third.html">
</frameset>
<noframes>
    <body>
        如浏览器不支持框架，才显示body内的内容。
    </body>
</noframes>
```

```html
<frameset cols="200,*,200" border="1" border-color="blue">
    <frame src="subframe/left.html">
    <frame src="subframe/center.html">
    <frame src="subframe/right.html">
</frameset>
```

在左侧实现导航，右侧显示内容的布局，使用`<a>`标签在左侧框架中链接右侧框架的内容：

```html
<frameset cols="20%,*" frameborder="0"> <!-- frameborder属性设置边框宽度 -->
  <frame src="subframe/nav.html" name="navFrame">
  <frame src="subframe/home.html" name="contentFrame">
</frameset>
```

> 注意：`<frameset>` 和 `<frame>` 在 HTML5 中已被废弃。建议使用 CSS（例如 Flexbox 或 Grid）进行页面布局，或在需要嵌入页面时使用 `<iframe>`（同时遵循目标站点的安全策略）。

```html
<!-- subframe/nav.html -->
<a href="page1.html" target="contentFrame">页面 1</a><br>
<a href="page2.html" target="contentFrame">页面 2</a><br>
<a href="page3.html" target="contentFrame">页面 3</a>
```

```html
<!-- page1.html -->
<h1>这是页面 1 的内容</h1>

<!-- page2.html -->
<h1>这是页面 2 的内容</h1>

<!-- page3.html -->
<h1>这是页面 3 的内容</h1>
```

### 内联框架(Iframe)

内联框架用于在当前页面中嵌入另一个HTML页面，使用`<iframe>`标签定义。  
与`<frameset>`不同，`<iframe>`可以与其他HTML元素共存。

```html
<body>
    <h1>主页面内容</h1>
    <iframe src="https://marlin-phone.github.io" width="50%" height="400">您的浏览器不支持内联框架。</iframe><br><br>
    <iframe src="https://marlin-phone.github.io/about/" width="50%" height="400" frameborder="0">您的浏览器不支持内联框架。</iframe>
</body>
```

<body>
    <h1>主页面内容</h1>
    <iframe src="https://marlin-phone.github.io" width="50%" height="400">您的浏览器不支持内联框架。</iframe><br><br>
    <iframe src="https://marlin-phone.github.io/about/" width="50%" height="400" frameborder="0">您的浏览器不支持内联框架。</iframe>
</body>