---
layout: post
mathjax: true
title: "HttpServletRequest与HttpServletResponse的区别"
subtitle: "HttpServletRequest and HttpServletResponse"
date: 2025-11-04 21:46:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - Servlet
---

# HttpServletRequest 与 HttpServletResponse 的区别

## 一、`HttpServletRequest` —— 封装客户端请求

### 📌 核心作用
获取**客户端发来的所有信息**：参数、请求头、会话、协议、路径等。

### ✅ 常用方法分类总结

| 类别                             | 方法                                       | 说明                            | 示例                                                   |
| -------------------------------- | ------------------------------------------ | ------------------------------- | ------------------------------------------------------ |
| **获取请求参数**                 | `String getParameter(String name)`         | 获取单个参数值（表单、URL）     | `request.getParameter("username")`                     |
|                                  | `String[] getParameterValues(String name)` | 获取多个同名参数（如 checkbox） | `request.getParameterValues("hobby")`                  |
|                                  | `Map<String, String[]> getParameterMap()`  | 获取所有参数（键值对）          | 用于日志或统一处理                                     |
| **获取请求属性（服务器端设置）** | `Object getAttribute(String name)`         | 获取 request 作用域的数据       | `request.getAttribute("user")`                         |
|                                  | `void setAttribute(String name, Object o)` | 设置属性（Servlet → JSP 传值）  | `request.setAttribute("msg", "OK")`                    |
|                                  | `void removeAttribute(String name)`        | 移除属性                        | —                                                      |
| **获取请求路径/URL 信息**        | `String getRequestURI()`                   | 请求路径（不含协议/主机）       | `/exp01/hello`                                         |
|                                  | `String getContextPath()`                  | 应用上下文路径                  | `/exp01`                                               |
|                                  | `String getServletPath()`                  | Servlet 路径                    | `/hello`                                               |
|                                  | `String getQueryString()`                  | URL 中 `?` 后的参数             | `name=marlin&age=20`                                   |
| **获取请求头**                   | `String getHeader(String name)`            | 获取某个请求头                  | `request.getHeader("User-Agent")`                      |
|                                  | `Enumeration<String> getHeaderNames()`     | 获取所有请求头名                | —                                                      |
| **获取会话（Session）**          | `HttpSession getSession()`                 | 获取或创建 session              | `request.getSession().setAttribute("loginUser", user)` |
|                                  | `HttpSession getSession(boolean create)`   | `false` 表示不自动创建          | 用于判断是否登录                                       |
| **获取客户端信息**               | `String getRemoteAddr()`                   | 客户端 IP 地址                  | `192.168.1.100`                                        |
|                                  | `String getMethod()`                       | 请求方法（GET/POST）            | `"GET"`                                                |
|                                  | `String getProtocol()`                     | HTTP 协议版本                   | `"HTTP/1.1"`                                           |

> ⚠️ 注意：
> - `getParameter()` 只能获取**客户端提交的数据**（表单、URL 参数）
> - `getAttribute()` 只能获取**服务器端 set 的数据**（仅当前请求有效，常用于 forward）

## 二、`HttpServletResponse` —— 封装服务器响应

### 📌 核心作用
**向客户端发送响应**：设置状态码、响应头、重定向、输出内容等。

### ✅ 常用方法分类总结

| 类别            | 方法                                        | 说明                           | 示例                                                 |
| --------------- | ------------------------------------------- | ------------------------------ | ---------------------------------------------------- |
| **发送重定向**  | `void sendRedirect(String location)`        | **302 重定向**（客户端跳转）   | `response.sendRedirect("login.html")`                |
| **设置响应头**  | `void setHeader(String name, String value)` | 设置普通响应头                 | `response.setHeader("Cache-Control", "no-cache")`    |
|                 | `void setContentType(String type)`          | 设置响应内容类型（MIME）       | `response.setContentType("text/html;charset=UTF-8")` |
|                 | `void setCharacterEncoding(String enc)`     | 设置字符编码                   | `response.setCharacterEncoding("UTF-8")`             |
| **获取输出流**  | `PrintWriter getWriter()`                   | 获取**字符输出流**（写文本）   | 用于输出 HTML、JSON                                  |
|                 | `ServletOutputStream getOutputStream()`     | 获取**字节输出流**（写二进制） | 用于下载文件、图片                                   |
|                 | ⚠️ 两者**不能同时用**！                      | —                              | —                                                    |
| **设置状态码**  | `void setStatus(int sc)`                    | 设置 HTTP 状态码               | `response.setStatus(404)`                            |
| **Cookie 操作** | `void addCookie(Cookie cookie)`             | 向客户端添加 Cookie            | `response.addCookie(new Cookie("theme", "dark"))`    |

> 💡 典型使用场景：
> ```java
> // 输出简单 HTML
> response.setContentType("text/html;charset=UTF-8");
> PrintWriter out = response.getWriter();
> out.println("<h1>你好</h1>");
> ```

## 三、对比记忆表（重点！）

| 功能       | `HttpServletRequest`                                           | `HttpServletResponse`               |
| ---------- | -------------------------------------------------------------- | ----------------------------------- |
| **方向**   | 客户端 → 服务器                                                | 服务器 → 客户端                     |
| **拿数据** | `getParameter()`（用户输入）<br>`getAttribute()`（服务器存的） | ❌ 不能拿客户端数据                  |
| **发数据** | ❌ 不能直接发                                                   | `getWriter()` / `getOutputStream()` |
| **跳转**   | 用 `RequestDispatcher.forward()`（服务器内部转发）             | 用 `sendRedirect()`（客户端重定向） |
| **作用域** | 可获取 request/session/application 属性                        | 不能直接操作作用域                  |

## 四、转发（forward） vs 重定向（redirect）速记

|                       | 转发（forward）                            | 重定向（redirect）        |
| --------------------- | ------------------------------------------ | ------------------------- |
| 谁做的？              | **服务器内部**                             | **浏览器重新请求**        |
| URL 变吗？            | ❌ 不变                                     | ✅ 变成新地址              |
| 能传 request 属性吗？ | ✅ 能（`getAttribute` 可用）                | ❌ 不能（新请求）          |
| 用哪个对象？          | `request.getRequestDispatcher().forward()` | `response.sendRedirect()` |