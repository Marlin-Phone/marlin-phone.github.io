---
layout: post
mathjax: true
title: "JavaScript 快速入门 1"
subtitle: "JavaScript Quick Start"
date: 2025-11-02 20:39:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - JavaScript
---

# JavaScript 快速入门

## JavaScript 基础语法

### 脚本的基本结构

```html
<script>
    // JavaScript 代码
</script>
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>标题</title>
    <script>
        // javascript 代码
        document.write("Hello, World!");
    </script>
</head>
<body>
    <!-- 页面主题内容 -->
</body>
</html>
<!-- ... -->
```

### JavaScript 变量与数据类型

#### 变量的声明与使用

在 JavaScript 中，变量用于存储数据值。可以使用 `var`、`let` 或 `const` 关键字来声明变量。

```js
// 使用 var 声明变量（函数作用域）
var x = 10;
if (true) {
    var y = 20;
}
console.log(x); // 10
console.log(y); // 20
// 使用 let 声明变量（块作用域）
let a = 30;
if (true) {
    let b = 40;
}
console.log(a); // 30
console.log(b); // ReferenceError: b is not defined
// 使用 const 声明常量（块作用域，值不可变）
const PI = 3.14;
console.log(PI); // 3.14
```

#### 数据类型

JavaScript 支持多种数据类型，主要包括：
- 原始数据类型：包括 `Number`、`String`、`Boolean`、`undefined` 和 `null`。
- 复杂数据类型：包括 `Object`、`Array` 和 `Function`。

```js
// 对象
let person = {
    name: "Alice",
    age: 25
};

// 数组
let colors = ["red", "green", "blue"];

// 函数
function greet() {
    console.log("Hello!");
}
```

`typeof` 运算符用于检查变量的数据类型，返回值为字符串：

- `"undefined"`：变量声明后，但未被赋值
- `"string"`：用单引号或双引号声明的字符串
- `"number"`：整数或浮点数
- `"boolean"`：true或false
- `"object"`：JavaScript中的对象、数组和null
- `"function"`：函数类型

```js
console.log(typeof x); // "number"
console.log(typeof person); // "object"
console.log(typeof greet); // "function"
```

### 基本语句与控制结构

#### 条件语句

```js
if (condition) {
    // 代码块
} else {
    // 代码块
}
```
```js
let age = 18;
if (age >= 18) {
    console.log("成年人");
} else {
    console.log("未成年人");
}
```

#### 循环语句

```js
for (initialization; condition; increment) {
    // 代码块
}
```

```js
for (let i = 0; i < 5; i++) {
    console.log(i);
}
```

可以使用`break`或`continue`来控制循环流程。

#### 注释

单行注释使用 `//`，多行注释使用 `/* ... */`。

```js
// 这是单行注释
/* 这是多行注释
   可以跨越多行 */
```

### 常用的输入/输出方法

`prompt()`：用于弹出一个对话框，提示用户输入信息，可以接收两个参数，第一个是提示信息，第二个是默认值（可选）。如果用户取消，返回值为 `null`。

```js
let name = prompt("请输入你的名字：", "游客");
if (name !== null) {
    alert("你好，" + name + "！");
} else {
    console.log('用户未输入名字（返回 null）');
}
```

### JavaScript 使用方式

- HTML 页面内嵌 JS 代码
- 外部 JS 文件引用
- 浏览器控制台执行

```html
<!-- HTML 页面内嵌JS代码 -->
<script>
    document.write("Hello, World!");
</script>
```

```html
<!-- 引入外部 JS 文件 -->
<script src="path/to/your/script.js"></script>
```

```js
// script.js
console.log("外部 JS 文件加载成功！");
```
