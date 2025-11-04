---
layout: post
mathjax: true
title: "Servlet 快速入门"
subtitle: "Servlet Quick Start"
date: 2025-11-03 23:51:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - Servlet
---

# Servlet 技术详解：JavaWeb 的核心控制器

## 一、Servlet 是什么？

### 定义与本质
**Servlet** 是运行在 Web 服务器或应用服务器上的 Java 程序，用于扩展服务器的功能，处理客户端（通常是浏览器）的请求并生成响应。其本质是一个**Java 接口**，定义了处理请求的标准方法。

### 关键特性
- **平台无关性**：基于 Java，可在任何支持 JVM 的服务器上运行
- **持久性**：Servlet 是单例的，加载后常驻内存，处理多个请求
- **高效性**：避免了 CGI 的进程创建开销，性能显著提升
- **可伸缩性**：可利用 Java 的多线程机制高效处理并发请求
- **安全性**：运行在服务器沙箱环境中，提供安全保障

## 二、Servlet 在 Web 架构中的位置

```
客户端 (浏览器)
       ↓
HTTP 请求
       ↓
Web 服务器 (如 Apache/Nginx)
       ↓
Servlet 容器 (如 Tomcat)
       ├───→ Servlet (处理业务逻辑)
       ├───→ JSP (生成视图)
       └───→ 其他资源
       ↓
HTTP 响应
       ↓
客户端 (浏览器)
```

- **Servlet 容器**（如 Tomcat 的 Catalina）：负责管理 Servlet 生命周期、处理请求分发
- **角色定位**：在 MVC 架构中，Servlet 通常充当 **Controller** 角色

## 三、Servlet 工作原理

### 请求处理流程
1. **客户端发起 HTTP 请求**（GET/POST 等）
2. **Web 服务器接收请求**，根据 URL 映射决定是否转发给 Servlet 容器
3. **Servlet 容器**：
   - 查找对应的 Servlet 实例
   - 若不存在，加载并实例化 Servlet
   - 调用 `service()` 方法处理请求
4. **Servlet 处理请求**：
   - 解析请求参数
   - 调用业务逻辑
   - 生成响应内容
5. **容器将响应返回**给 Web 服务器，最终返回给客户端

### 核心组件关系
```
                 +-----------------+
                 |   Web 服务器    |
                 | (Apache/Nginx)  |
                 +--------+--------+
                          |
                 +--------v--------+
                 |  Servlet 容器   |
                 |    (Tomcat)     |
                 +--------+--------+
                          |
          +---------------+---------------+
          |               |               |
+---------v------+ +------+-------+ +-----v--------+
|   HTTPServlet  | |  GenericServlet| | 其他Servlet  |
+----------------+ +--------------+ +-------------+
          |
+---------v------+
| 开发者自定义Servlet|
+----------------+
```

## 四、Servlet 生命周期

Servlet 的生命周期由容器管理，分为三个阶段：

### 1. 初始化阶段（Initialization）
- **触发时机**：Servlet 首次被请求时，或容器启动时（配置了 `<load-on-startup>`）
- **关键方法**：`public void init(ServletConfig config) throws ServletException`
- **执行内容**：
  - 加载配置参数
  - 初始化数据库连接池
  - 加载缓存数据
  - 一次性资源准备
- **特点**：整个生命周期中只执行一次

### 2. 服务阶段（Service）
- **触发时机**：每次接收到 HTTP 请求
- **关键方法**：`public void service(ServletRequest req, ServletResponse res)`
- **执行流程**：
  - 容器创建 `HttpServletRequest` 和 `HttpServletResponse` 对象
  - 根据 HTTP 方法（GET/POST/PUT/DELETE）调用相应方法
  - 在多线程环境下执行（**线程不安全**！）
- **特点**：可执行多次，是处理业务的核心阶段

### 3. 销毁阶段（Destruction）
- **触发时机**：容器关闭或应用卸载时
- **关键方法**：`public void destroy()`
- **执行内容**：
  - 释放数据库连接
  - 保存缓存数据
  - 释放其他资源
- **特点**：整个生命周期中只执行一次

## 五、核心接口与类

### 1. 核心接口

| 接口                             | 说明                         |
| -------------------------------- | ---------------------------- |
| `javax.servlet.Servlet`          | 最顶层接口，定义生命周期方法 |
| `javax.servlet.ServletConfig`    | 提供 Servlet 配置信息        |
| `javax.servlet.ServletContext`   | 提供应用上下文信息           |
| `javax.servlet.ServletRequest`   | 封装客户端请求信息           |
| `javax.servlet.ServletResponse`  | 封装服务器响应信息           |
| `javax.servlet.http.HttpServlet` | 处理 HTTP 请求的抽象类       |

### 2. 常用实现类

| 类                           | 说明                   | 典型用途            |
| ---------------------------- | ---------------------- | ------------------- |
| `HttpServlet`                | 抽象类，实现 HTTP 协议 | 所有 Web 应用的基类 |
| `GenericServlet`             | 通用协议 Servlet       | 非 HTTP 协议场景    |
| `HttpServletRequestWrapper`  | 请求包装类             | 实现过滤、修改请求  |
| `HttpServletResponseWrapper` | 响应包装类             | 实现过滤、修改响应  |

## 六、核心方法详解

### 1. HTTP 方法处理
`HttpServlet` 为不同 HTTP 方法提供了专用方法：

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) // 处理 GET 请求
protected void doPost(HttpServletRequest req, HttpServletResponse resp) // 处理 POST 请求
protected void doPut(HttpServletRequest req, HttpServletResponse resp) // 处理 PUT 请求
protected void doDelete(HttpServletRequest req, HttpServletResponse resp) // 处理 DELETE 请求
```

### 2. 关键方法功能

| 方法        | 功能           | 注意事项                     |
| ----------- | -------------- | ---------------------------- |
| `init()`    | 初始化资源     | 避免长时间操作，阻塞容器启动 |
| `service()` | 请求分发       | 通常不重写，由容器调用       |
| `doGet()`   | 处理 GET 请求  | 参数在 URL 中，长度有限      |
| `doPost()`  | 处理 POST 请求 | 适合表单提交，数据量大       |
| `destroy()` | 释放资源       | 确保所有资源被正确关闭       |

### 3. 请求/响应处理
```java
// 获取请求参数
String username = request.getParameter("username");
String[] hobbies = request.getParameterValues("hobby");

// 设置响应内容类型
response.setContentType("text/html;charset=UTF-8");

// 获取输出流
PrintWriter out = response.getWriter();
out.println("<h1>Hello, " + username + "!</h1>");

// 重定向
response.sendRedirect("success.html");

// 请求转发
request.getRequestDispatcher("result.jsp").forward(request, response);

// 设置 Cookie
Cookie userCookie = new Cookie("username", username);
userCookie.setMaxAge(3600); // 1小时
response.addCookie(userCookie);

// 获取 Session
HttpSession session = request.getSession();
session.setAttribute("user", user);
```

## 七、Servlet 配置方式

### 1. 传统方式：web.xml 配置
```xml
<servlet>
    <servlet-name>LoginServlet</servlet-name>
    <servlet-class>com.example.LoginServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>LoginServlet</servlet-name>
    <url-pattern>/login</url-pattern>
</servlet-mapping>
```

### 2. 注解方式（Servlet 3.0+）
```java
@WebServlet(
    name = "LoginServlet",
    urlPatterns = {"/login", "/signin"},
    loadOnStartup = 1,
    initParams = {
        @WebInitParam(name = "maxAttempts", value = "3")
    }
)
public class LoginServlet extends HttpServlet {
    // 实现代码
}
```

### 3. 编程式配置（Servlet 3.0+）
```java
public class MyServletRegistration implements ServletContainerInitializer {
    @Override
    public void onStartup(Set<Class<?>> c, ServletContext ctx) {
        ServletRegistration.Dynamic reg = 
            ctx.addServlet("DynamicServlet", new DynamicServlet());
        reg.addMapping("/dynamic");
        reg.setLoadOnStartup(2);
    }
}
```

## 八、线程安全问题

### 问题根源
- Servlet 实例是单例的（Singleton）
- 多个请求共享同一个 Servlet 实例
- 成员变量在多线程环境下可能产生数据不一致

### 解决方案
1. **避免使用实例变量**
   ```java
   // 不安全
   private String username; // 实例变量
   
   // 安全
   protected void doGet(HttpServletRequest req, HttpServletResponse res) {
       String username = req.getParameter("username"); // 局部变量
   }
   ```

2. **使用同步机制**
   ```java
   private final Object lock = new Object();
   
   protected void doPost(HttpServletRequest req, HttpServletResponse res) {
       synchronized(lock) {
           // 临界区代码
       }
   }
   ```

3. **使用线程安全对象**
   ```java
   private final ConcurrentHashMap<String, User> userCache = new ConcurrentHashMap<>();
   ```

4. **使用 ThreadLocal**
   ```java
   private static final ThreadLocal<SimpleDateFormat> dateFormatHolder = 
       ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
   ```

## 九、过滤器与监听器

### 1. 过滤器（Filter）
- **作用**：在请求到达 Servlet 前/后处理请求/响应
- **应用场景**：编码转换、权限检查、日志记录、压缩响应
- **核心接口**：`javax.servlet.Filter`
- **示例**：
  ```java
  @WebFilter("/*")
  public class CharacterEncodingFilter implements Filter {
      public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) {
          req.setCharacterEncoding("UTF-8");
          res.setContentType("text/html;charset=UTF-8");
          chain.doFilter(req, res); // 传递给下一个过滤器或Servlet
      }
  }
  ```

### 2. 监听器（Listener）
- **作用**：监听 Web 应用中的关键事件
- **常见类型**：
  - `ServletContextListener`：监听应用启动/销毁
  - `HttpSessionListener`：监听会话创建/销毁
  - `ServletRequestListener`：监听请求开始/结束
- **示例**：
  ```java
  @WebListener
  public class AppContextListener implements ServletContextListener {
      public void contextInitialized(ServletContextEvent sce) {
          // 应用启动时初始化数据库连接池
          DataSource ds = createDataSource();
          sce.getServletContext().setAttribute("dataSource", ds);
      }
      
      public void contextDestroyed(ServletContextEvent sce) {
          // 应用关闭时释放资源
          DataSource ds = (DataSource)sce.getServletContext().getAttribute("dataSource");
          if (ds != null) ds.close();
      }
  }
  ```

## 十、Servlet 3.0+ 新特性

### 1. 注解配置
- 消除 web.xml 配置负担
- `@WebServlet`, `@WebFilter`, `@WebListener` 等注解

### 2. 异步处理
- 解决长轮询、实时通知等场景
- 释放请求线程，提高容器吞吐量
- **核心方法**：
  ```java
  @WebServlet(urlPatterns = "/async", asyncSupported = true)
  public class AsyncServlet extends HttpServlet {
      protected void doGet(HttpServletRequest req, HttpServletResponse res) {
          AsyncContext asyncCtx = req.startAsync();
          asyncCtx.setTimeout(30000);
          
          new Thread(() -> {
              // 模拟耗时操作
              try { Thread.sleep(10000); } catch (Exception e) {}
              
              // 完成异步处理
              asyncCtx.getResponse().getWriter().println("异步处理完成");
              asyncCtx.complete();
          }).start();
      }
  }
  ```

### 3. 文件上传 API
- 简化文件上传处理
- 内置对 multipart/form-data 的支持
  ```java
  @MultipartConfig
  public class FileUploadServlet extends HttpServlet {
      protected void doPost(HttpServletRequest req, HttpServletResponse res) {
          Part filePart = req.getPart("file");
          String fileName = Paths.get(filePart.getSubmittedFileName()).getFileName().toString();
          
          // 保存文件
          filePart.write("/uploads/" + fileName);
      }
  }
  ```

### 4. 动态注册
- 运行时动态添加 Servlet、Filter、Listener
- 适合插件化架构

## 十一、实战示例：用户注册功能

### 1. 创建 Servlet
```java
@WebServlet("/register")
public class RegisterServlet extends HttpServlet {
    private UserService userService;
    
    @Override
    public void init() throws ServletException {
        // 通常应该使用依赖注入，简化示例
        this.userService = new UserServiceImpl();
    }
    
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        
        // 1. 设置编码
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");
        
        // 2. 获取参数
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String email = request.getParameter("email");
        
        // 3. 验证输入
        if (username == null || username.trim().isEmpty()) {
            request.setAttribute("error", "用户名不能为空");
            request.getRequestDispatcher("/register.jsp").forward(request, response);
            return;
        }
        
        // 4. 处理业务
        try {
            User user = userService.register(username, password, email);
            
            // 5. 保存到会话
            HttpSession session = request.getSession();
            session.setAttribute("currentUser", user);
            
            // 6. 重定向到主页（防止表单重复提交）
            response.sendRedirect(request.getContextPath() + "/home");
            
        } catch (DuplicateUserException e) {
            request.setAttribute("error", "用户名已存在");
            request.getRequestDispatcher("/register.jsp").forward(request, response);
        } catch (Exception e) {
            request.setAttribute("error", "注册失败: " + e.getMessage());
            request.getRequestDispatcher("/register.jsp").forward(request, response);
        }
    }
}
```

### 2. web.xml 配置（如使用传统方式）
```xml
<servlet>
    <servlet-name>RegisterServlet</servlet-name>
    <servlet-class>com.example.servlet.RegisterServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>RegisterServlet</servlet-name>
    <url-pattern>/register</url-pattern>
</servlet-mapping>
```

## 十二、最佳实践

### 1. 关注点分离
- **Controller 层 (Servlet)**：只处理请求分发，不包含业务逻辑
- **Service 层**：实现核心业务逻辑
- **DAO 层**：负责数据访问
- **View 层 (JSP/Thymeleaf)**：只负责显示

### 2. RESTful 设计
- 使用 HTTP 方法语义：GET(获取)、POST(创建)、PUT(更新)、DELETE(删除)
- 设计有意义的 URL：`/users`, `/users/123`
- 使用状态码表示结果：200(OK)、201(Created)、404(Not Found)、500(Internal Error)

### 3. 异常处理
- 全局异常处理（使用过滤器或错误页面配置）
- 避免向客户端暴露内部错误详情
- 记录详细错误日志

### 4. 安全实践
- 防止 XSS：对输出进行编码
- 防止 CSRF：使用 token 验证
- 防止 SQL 注入：使用预编译语句
- 敏感操作验证权限

## 十三、现代 Web 开发中的定位

### 传统应用
- 与 JSP 配合构建 MVC 架构
- 适合中后台管理系统

### 现代应用
- **API 服务**：作为 REST API 的控制器
- **微服务**：在 Spring Boot 等框架中作为基础组件
- **前后端分离**：提供 JSON 数据，前端使用 Vue/React 等框架

### 框架演进
- **Servlet → Spring Web MVC → Spring Boot → Spring WebFlux**
- 虽然框架在演进，但底层仍基于 Servlet API
- 了解 Servlet 有助于理解上层框架的工作原理

## 总结

Servlet 作为 Java Web 开发的核心技术，是理解整个 Java Web 生态的基础。通过掌握其工作原理、生命周期和最佳实践，您将能够：

1. 构建高性能、可维护的 Web 应用
2. 理解上层框架（如 Spring MVC）的工作原理
3. 解决复杂的 Web 开发问题
4. 设计符合行业标准的架构模式

> **提示**：在实际开发中，很少直接使用原始 Servlet API 构建应用，而是使用 Spring Boot 等框架简化开发。但深入理解 Servlet 对于解决问题、性能优化和架构设计至关重要。

**建议学习路径**：Servlet 基础 → Filter/Listener → JSP/EL/JSTL → JDBC 数据库连接 → MVC 架构 → Spring Boot