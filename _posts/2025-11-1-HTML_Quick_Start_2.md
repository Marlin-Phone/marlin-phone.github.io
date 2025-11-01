---
layout: post
mathjax: true
title: "HTML 快速入门 2"
subtitle: "HTML Quick Start"
date: 2025-11-01 15:22:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - HTML
---

# HTML 快速入门

## HTML 核心标签与用法

### 超链接

```html
<a href="链接地址" target="目标窗口位置"> 链接热点文本或图像 </a>
```

`href`全称为超文本链接（Hypertext Reference），可以为绝对地址，也可以为相对地址。

例子：
```html
<a href="https://marlin-phone.github.io/2025/10/31/HTML_Quick_Start_1/">HTML 快速入门1</a>
<a href="../../10/31/HTML_Quick_Start_1/">HTML 快速入门1</a>
```
<a href="https://marlin-phone.github.io/2025/10/31/HTML_Quick_Start_1/">HTML 快速入门1</a><br>
<a href="../../../10/31/HTML_Quick_Start_1/">HTML 快速入门1</a>

target 可以为：
- `_self`（默认值）：在当前窗口中打开链接。
- `_blank`：在新窗口或标签页中打开链接。
- `_parent`：在父级上下文中打开链接（例如嵌套的 `<iframe>` 中）。
- `_top`：在整个浏览器窗口中打开链接，跳出嵌套上下文。
- `framename`：在指定的 `<iframe>` 或命名的窗口中打开链接（需指定目标名称）.

例子：

```html
<a href="https://github.com/marlin-phone">我的 github 主页</a>
<a href="https://github.com/marlin-phone" target="_blank">我的 github 主页</a>
<a href="https://github.com/marlin-phone" target="_parent">我的 github 主页</a>
<a href="https://github.com/marlin-phone" target="_top">我的 github 主页</a>
<a href="https://github.com/marlin-phone" target="_framename">我的 github 主页</a>
```

<a href="https://github.com/marlin-phone">我的 github 主页</a>  
<a href="https://github.com/marlin-phone" target="_blank">我的 github 主页</a>  
<a href="https://github.com/marlin-phone" target="_parent">我的 github 主页</a>  
<a href="https://github.com/marlin-phone" target="_top">我的 github 主页</a>  
<a href="https://github.com/marlin-phone" target="_framename">我的 github 主页</a>

#### 锚链接(Anchor Link)

锚链接的组成：

1. 目标锚点：
    - 使用`id`属性定义页面中的锚点。
    - 示例：
        ```html
        <h2 id="end">这是结尾</h2>
        ``` 
2. 锚链接
    - 使用`href`属性指定目标锚点。
    - 示例：
        ```html
        <a href="#end">跳转到结尾</a>
        ```

<a href="#end">跳转到结尾</a>

页面间锚链接

1. **页面间锚链接**：
   - `href="anchor.html#star"` 表示跳转到 `anchor.html` 页面中 `id="star"` 的位置。
   - 目标页面 `anchor.html` 中需要定义对应的锚点，例如：`<h2 id="star">明星专区内容</h2>`。
   - 示例：
      ```html
      <!-- 当前页面 -->
      <a href="anchor.html#star">明星专区</a>

      <!-- 目标页面 anchor.html -->
      <h2 id="star">明星专区内容</h2>
      ```
2. **页面内锚链接**：
   - 如果锚点在当前页面，可以直接使用 `href="#star"`。
   - 示例：
      ```html
      <!-- 当前页面 -->
      <a href="#star">跳转到明星专区</a>

      <h2 id="star">明星专区内容</h2>
      ```

### 注释

```html
<!-- 这里是注释，不会显示在页面中。 -->
```

### 特殊符号

1. 因为`<`、`>`等符号在HTML中已使用，所以必须用其他符号来代替。
2. 都以分号结束(;)

```html
空格:&nbsp;
大于(>):&gt;
小于(<):&lt;
引号("):&quot;
版权号:&copy;
```

示例：

空格:&nbsp;  
大于(>):&gt;  
小于(<):&lt;  
引号("):&quot;  
版权号:&copy;

### 表单(Form)

表单是一个包含表单元素的区域。

```html
<form action="表单提交地址" method="提交方法">
  <!-- 文本框、按钮等表单元素 -->
</form>
```

action:指定提交后由服务器上哪个程序处理。  
method:指定向服务器提交的方法，如`get`、`post`，默认为`get`。


#### 表单元素

```html
<input name="表单元素名称" type="类型" value="值" size="显示宽度" maxlength="最大长度" checked="是否选中">
```

name:控件的名称。

type:指定元素的类型，可以为`text`、`password`、`checkbox`、`radio`、`button`、`image`、`submit`、`file`等。  
文本框`text`：用于输入单行文本。  
<input name="exampleText" type="text" value="示例文本" size="20" maxlength="50">  
密码框`password`：用于输入密码，输入内容以掩码显示。  
<input name="examplePassword" type="password" value="mypassword" size="20" maxlength="50">  
复选框`checkbox`：用于输入多项选择，用户可以选择多个选项。  
<input name="exampleCheckbox" type="checkbox" value="option1">足球  
<input name="exampleCheckbox" type="checkbox" value="option2">篮球  
<input name="exampleCheckbox" type="checkbox" value="option3">排球  
单选框`radio`：用于输入单项选择，用户只能选择一个选项。  
<input name="exampleRadio" type="radio" value="option1" checked>男  
<input name="exampleRadio" type="radio" value="option2">女  
按钮`button`：用于自定义按钮，通常配合JavaScript使用。  
<input name="exampleButton" type="button" value="点击我">  
图片按钮`image`：用于提交表单的图片按钮。  
<input name="exampleImage" type="image" src="button.png" alt="提交">  
提交按钮`submit`：用于提交表单。  
<input name="exampleSubmit" type="submit" value="提交表单">  
文件上传框`file`：用于上传文件。  
<input name="exampleFile" type="file">

value:控件的初始值。  
<input name="exampleText" type="text" value="初始值">

size:控件的初始宽度。  
<input name="exampleText" type="text" size="30">

maxlength:控件中最多输入的字符数量。  
<input name="exampleText" type="text" maxlength="1">

checked:单选框或复选框是否被选中。  
<input name="exampleCheckbox" type="checkbox" checked>

```html
<form action="login.jsp" method="post">
  <p>用户名: <input name="username" type="text" size="20" maxlength="32"></p>
  <p>密&nbsp;&nbsp;码：<input name="password" type="password" size="20" maxlength="32"></p>
  <p><input type="submit" value="提交">&nbsp;&nbsp;<input type="reset" value="重填"></p>
</form>
```

<form action="login.jsp" method="post">
  <p>用户名: <input name="username" type="text" size="20" maxlength="32"></p>
  <p>密&nbsp;&nbsp;码：<input name="password" type="password" size="20" maxlength="32"></p>
  <p><input type="submit" value="提交">&nbsp;&nbsp;<input type="reset" value="重填"></p>
  <p><input name="exampleFile" type="file"></p>
</form>

列表框(select)

```html
<select name="bmonth">
  <option value="" selected="selected">[选择月份]</option>
  <option value="1">一月</option>
  <option value="2">二月</option>
  <option value="3">三月</option>
</select>
```
<select name="bmonth">
  <option value="" selected="selected">[选择月份]</option>
  <option value="1">一月</option>
  <option value="2">二月</option>
  <option value="3">三月</option>
</select>

多行文本框(textarea)

```html
<textarea name="..." rows="行高" cols="列宽">文本内容</textarea>
```

```html
<textarea name="comments" rows="5" cols="30">在此输入内容...</textarea>
```

<textarea name="comments" rows="5" cols="30">在此输入内容...</textarea>

隐藏域(hidden)

方便服务端接收一些不需要用户看到的数据。

```html
<input name="hiddenField" type="hidden" value="隐藏值">
```
<input name="hiddenField" type="hidden" value="隐藏值">

只读和禁用属性(readonly 和 disabled)

```html
<textarea name="readonlyField" rows="5" cols="30" readonly>欢迎阅读服务条款</textarea><br>
同意以上协议<input name="agreement" type="checkbox">
<input name="disabledField" type="submit" value="提交" disabled>
<!-- 通过JavaScript可以实现“确认按钮选中后才能点击提交”的功能 -->
```

<textarea name="readonlyField" rows="5" cols="30" readonly>欢迎阅读服务条款</textarea><br>
同意以上协议<input name="agreement" type="checkbox">  
<input name="disabledField" type="submit" value="提交" disabled>

<h2 id="end">这是结尾</h2>