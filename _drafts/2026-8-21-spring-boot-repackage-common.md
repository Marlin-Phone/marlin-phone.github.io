---
layout: post
mathjax: true
title: "Spring Boot 多模块编译踩坑：公共库模块的 repackage 问题"
subtitle: "Unable to find main class 的终极解法"
date: 2026-8-21 03:15:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - Spring Boot
  - Maven
  - 踩坑
---

> ⚠️ **AI 生成标识** · 本文由 AI Agent 协助撰写，人类作者审核发布。

## 背景

在部署一个 Spring Boot 多模块项目时，执行：

```bash
mvn -pl device-health-data-connection -am package -DskipTests
```

编译报错：

```
[ERROR] Failed to execute goal org.springframework.boot:spring-boot-maven-plugin:3.5.6:repackage
(repackage) on project device-health-common: Execution repackage of goal
org.springframework.boot:spring-boot-maven-plugin:3.5.6:repackage failed:
Unable to find main class
```

## 原因分析

在 Spring Boot 多模块项目中，**公共库模块**（如 `common`、`client`）本身不是可执行应用，没有 `main` 方法。

但父 POM 中配置了 `spring-boot-maven-plugin`，该插件默认会尝试把每个模块 **repackage 成可执行 jar**（fat jar）。当它遇到没有 `main` 类的模块时，就会报 `Unable to find main class`。

## 解决方案

给公共库模块的 `spring-boot-maven-plugin` 添加 `<skip>true</skip>`：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <skip>true</skip>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**或者**，更推荐的做法是在父 POM 的 `pluginManagement` 中统一配置，排除公共库：

```xml
<pluginManagement>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</pluginManagement>
```

> 💡 但注意：`pluginManagement` 里的 `<skip>` 会全局生效，**所有模块都不会被 repackage**，可执行模块也会受影响。所以正确做法是：**只在公共库模块的 build 中显式加 `<skip>true</skip>`**。

## 判断标准

哪些模块需要加 `<skip>true</skip>`？

| 模块类型 | 是否有 main 类 | 需要 skip |
|:---------|:---------------|:----------|
| 应用模块（Application） | ✅ 有 | ❌ 不需要 |
| 公共库（common/utils/constants） | ❌ 无 | ✅ 需要 |
| 客户端库（client/feign） | ❌ 无 | ✅ 需要 |

> 判断技巧：看模块里有没有 `@SpringBootApplication` 注解的类。

## 效果

修复后编译正常：

```
[INFO] BUILD SUCCESS
```

生成的可执行 jar 也符合预期（约 142MB）。

## 避坑提醒

1. **报错信息是误导性的**：`Unable to find main class` 不是真的"找不到主类"，而是"这个模块本来就不该有主类，但插件非要找"
2. **不要全局 skip**：`pluginManagement` 里统一加 `<skip>true</skip>` 会影响所有模块，可执行 jar 也会变成普通 jar
3. **常见于多模块项目**：如果项目里只有一个模块，不会遇到这个问题；多模块 + 公共库时才会爆