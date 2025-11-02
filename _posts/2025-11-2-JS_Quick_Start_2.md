---
layout: post
mathjax: true
title: "JavaScript 快速入门 2"
subtitle: "JavaScript Quick Start"
date: 2025-11-02 23:12:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - JavaScript
---

# JavaScript 快速入门

## JavaScript 函数和对象

### 函数的定义

函数是执行特定任务或计算的可重用代码块。可以使用 `function` 关键字来定义函数。

```js
function 函数名(参数1, 参数2, ...) {
    // 函数体
    return 返回值; // 可选
}
```

### JavaScript 核心

#### 系统函数

- `parseInt()`：将字符串解析为整数。
- `parseFloat()`：将字符串解析为浮点数。
- `isNaN()`：检查一个值是否为非数字（NaN）。
- `eval(String)`：计算字符串表达式并返回结果。
- `Number(Object)`：将对象转换为数字类型。
- `String(Object)`：将对象转换为字符串类型。

#### 自定义函数

创建函数：

无参函数
```js
function 函数名(){
    JavaScript 代码;
}
```

有参函数
```js
function 函数名(参数1, 参数2, ...){
    JavaScript 代码;
}
```

函数的调用

函数调用一般和表单元素的事件一起使用，调用格式：
```js
事件名 = "函数名()";
```

示例：
```js
function showMessage(){
    // document.write() 方法用于向 HTML 文档写入内容，会覆盖现有内容。
    // 已经不推荐使用，此处仅做演示。
    document.write("Hello, World!"); 
}
```

```html
<input type="button" value="点击显示信息" onclick="showMessage()">
```

### Window 对象

常用属性：

| 属性/对象        | 说明                                                                                                                                                                 |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| frames           | 如果 Window 中包含帧，则 frames 为一个数组对象，保存每个帧对应的子窗口。可以通过 window.frames["frame-name"]或数组下标 0..n 来访问。                                 |
| opener           | 返回对创建此窗口的窗口的引用。                                                                                                                                       |
| name             | 设置或返回存放窗口的名称的一个字符串。窗口的名称可以用作一个 `<a>` 或者 `<form>` 标记的 target 属性的值。                                                            |
| **window, self** | Window 和 self 均为对当前窗口的引用，相当于 this。                                                                                                                   |
| **top**          | 返回顶级窗口的只读引用。                                                                                                                                             |
| closed           | 返回一个布尔值，该值声明了窗口是否已经关闭。该属性为只读。当浏览器窗口关闭时，表示该窗口的 Windows 对象并不会消失，它将继续存在，不过它的 closed 属性将设置为 true。 |
| location         | Location 对象是 Window 对象的一个部分，包含有关当前 URL 的信息。可通过 window.location 属性来访问。                                                                  |
| history          | History 对象是 window 对象的一部分，包含用户在当前浏览器窗口中访问过的 URL。                                                                                         |
| document         | 每个载入浏览器的 HTML 文档都会成为 Document 对象，Document 对象是 Window 对象的一部分，Document 对象使我们可以从脚本中对 HTML 页面中的所有元素进行访问。             |
| defaultStatus    | 设置或返回窗口状态栏中的默认文本，该属性可读可写。                                                                                                                   |
| status           | 设置或返回窗口状态栏中的文本。                                                                                                                                       |
| navigator        | navigator 对象包含有关浏览器的信息。                                                                                                                                 |
| screen           | 包含有关客户端显示屏幕的信息。                                                                                                                                       |

示例：
```js
// 确保该网站不被嵌入在其他网站的框架中，被嵌入的话则跳转到该网站。
// google/baidu 等大型网站常用此方法类防止点击劫持攻击。
if (window.top !== window.self) {
    window.top.location = window.self.location;
}
```

常用方法：

| 方法          | 说明                                       |
| ------------- | ------------------------------------------ |
| prompt()      | 显示一个对话框，提示用户输入信息。         |
| alert()       | 显示一个警告对话框，向用户传递信息。       |
| confirm()     | 显示一个对话框，向用户传递信息并要求确认。 |
| close()       | 关闭当前窗口。                             |
| open()        | 打开一个新窗口。                           |
| setTimeout()  | 在指定的毫秒数后调用函数或计算表达式一次。 |
| setInterval() | 每隔指定的毫秒数调用函数或计算表达式       |

confirm() 方法：
```js
let flag = confirm("你确定要删除此条信息吗？");
if (flag) {
    alert("删除成功！")
} else {
    alert("你取消了删除。");
}
```

confirm()与alert()、prompt()的区别：
- `alert()` 仅显示信息，用户点击“确定”后关闭对话框。
- `prompt()` 提示用户输入信息，并返回用户输入的值或 `null`（如果取消）。
- `confirm()` 提示用户确认操作，返回布尔值 `true`（点击“确定”）或 `false`（点击“取消”）。

open() 方法：

window.open("弹出窗口的 URL", "窗口名称", "窗口特征");

窗口特征参数说明：
| 名称                          | 说明                                                                           |
| ----------------------------- | ------------------------------------------------------------------------------ |
| height、width                 | 窗口文档显示区的高度、宽度。以像素计。                                         |
| left、top                     | 窗口的x坐标、y坐标。以像素计                                                   |
| toolbar=yes \| no \|1 \| 0    | 是否显示浏览器的工具栏。默认是yes。                                            |
| scrollbars=yes \| no \|1 \| 0 | 是否显示滚动条。默认是yes。                                                    |
| location=yes \| no \|1 \| 0   | 是否显示地址地段。默认是yes。                                                  |
| status=yes \| no \|1 \| 0     | 是否添加状态栏。默认是yes。                                                    |
| menubar=yes \| no \|1 \| 0    | 是否显示菜单栏。默认是yes。                                                    |
| resizable=yes \| no \|1 \| 0  | 窗口是否可调节尺寸。默认是yes。                                                |
| titlebar=yes \| no \|1 \| 0   | 是否显示标题栏。默认是yes。                                                    |
| fullscreen=yes \| no \|1 \| 0 | 是否使用全屏模式显示浏览器。默认是no。处于全屏模式的窗口必须同时处于剧院模式。 |

### Window 窗口的常用事件

| 名称        | 说明                               |
| ----------- | ---------------------------------- |
| onload      | 一个页面或一幅图像完成加载         |
| onmouseover | 鼠标移到某元素之上                 |
| onclick     | 当用户单击某个对象时调用的事件句柄 |
| onkeydown   | 某个键盘按键被按下                 |
| onchange    | 域的内容被改变                     |

### 匿名调用函数

匿名函数是没有名称的函数，通常用于需要临时使用的场景。可以将匿名函数赋值给变量，或作为参数传递给其他函数。

语法：
```js
const 函数名 = function(参数) {
    // 函数体
};
```

示例：
```js
const greet = function(name) {
    return "Hello, " + name + "!";
};
```

### Date 对象

Date 对象用于处理日期和时间。可以创建一个 Date 对象来表示当前日期和时间，或者指定一个特定的日期和时间。
创建 Date 对象：
```js
const now = new Date(); // 当前日期和时间
const specificDate = new Date("2025-11-02T12:00:00"); // 指定日期和时间
```
| Date 方法分组 | 说明                                                           |
| ------------- | -------------------------------------------------------------- |
| setXxx()      | 用于设置日期和时间的各个部分，如年、月、日、小时、分钟、秒等。 |
| getXxx()      | 用于获取日期和时间的各个部分，如年、月、日、小时、分钟、秒等。 |

常用方法：
| 值               | 整数                            |
| ---------------- | ------------------------------- |
| Seconds和Minutes | 0-59                            |
| Hours            | 0-23                            |
| Day              | 0-6（0表示星期天，6表示星期六） |
| Date             | 1-31                            |
| Months           | 0-11（0表示一月，11表示十二月） |
| FullYear         | 四位年份，如2025                |

示例：
```js
function displayDateTime() {
    const now = new Date();
    console.log("当前年份：" + now.getFullYear());
    console.log("当前月份：" + (now.getMonth() + 1)); // 月份从0开始计数
    console.log("当前日期：" + now.getDate());
    console.log("当前时间：" + now.getHours() + ":" + now.getMinutes() + ":" + now.getSeconds());
}
```

setTimeout() 和 setInterval() 方法：
- setTimeout() 用于在指定的延迟后执行代码一次
- setInterval() 用于每隔指定的时间间隔重复执行代码。

```js
setTimeout(function() { // 延迟1秒后执行
    displayDateTime();
}, 1000);

setInterval(function() { // 每隔1秒执行一次
    displayDateTime();
}, 1000);
```