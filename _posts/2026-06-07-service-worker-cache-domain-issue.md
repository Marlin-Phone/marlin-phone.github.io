---
layout: post
title: "一个域名，两个面孔：记一次 Service Worker 缓存引发的灵异事件"
subtitle: "排查博客自定义域名后 github.io 和 marlin-phone.me 显示不同内容的完整过程"
date: 2026-06-07
author: Marlin
header-img: img/home-bg.jpg
tags:
  - 博客
  - GitHub Pages
  - Service Worker
  - DNS
  - 故障排查
---

> 本文使用了AI进行辅助写作，旨在提供更清晰、结构化的内容。AI生成的部分已由作者审核和编辑，以确保准确性和可读性。

## 背景

我的博客托管在 GitHub Pages，最近给它配了自定义域名 `marlin-phone.me`，通过 Cloudflare 做 CDN 代理。一切配置看起来都正常——DNS 解析正确、Cloudflare SSL 配置完整、GitHub 仓库里 CNAME 文件也在。

直到有一天，我发现了一个诡异的现象：

- 访问 **marlin-phone.me** → ✅ 最新文章、最新样式
- 访问 **marlin-phone.github.io** → ❌ 旧版页面、缺少最近发布的文章

同一个仓库、同一个分支（master）、同一份代码，为什么两个域名会显示**不同的内容**？

这就开始了我的排查之旅。

---

## 第一步：怀疑重定向

GitHub Pages 对自定义域名的标准行为是：访问 `github.io` → 301 重定向 → 自定义域名。

我用 `curl` 追踪了一下重定向链：

```bash
$ curl -sIL http://marlin-phone.github.io

HTTP/1.1 301 → Location: http://marlin-phone.me/    # ← 注意：HTTP！
HTTP/1.1 301 → Location: https://marlin-phone.me/
HTTP/1.1 200 OK
```

第一个发现：**GitHub 的重定向指向 `http://` 而不是 `https://`**。

这是因为我的域名 DNS 托管在 Cloudflare（开启了橙色云朵代理），TLS 终止在 Cloudflare 层。GitHub 看不到我的域名有 HTTPS 证书，保守起见就重定向到了 HTTP。

但 Cloudflare 的「Always Use HTTPS」开着，收到的 HTTP 请求又被 301 到了 HTTPS。所以这个链路上应该没问题。

**重定向链路正常，但问题依然存在。第一怀疑排除。**

---

## 第二步：怀疑仓库分支

是不是 GitHub Pages 部署了不同分支？

```bash
$ curl https://api.github.com/repos/Marlin-Phone/.../branches
["copilot/fix-github-actions-build-error", "master"]
```

只有 `master` 分支（没有 `gh-pages`），所有 71 篇文章都在 `_posts/` 下。仓库源码和自定义域名展示的内容理论上完全一致。

**仓库结构没问题。第二怀疑排除。**

---

## 第三步：怀疑 `_config.yml` 的 `url` 配置

```yaml
url: "https://marlin-phone.github.io"    # ← 还是指向 github.io！
```

确实应该改成 `marlin-phone.me`，但这只影响 RSS feed、sitemap、canonical URL 等 SEO 相关的元数据，**不会影响页面渲染内容**。

**不是根因，但属于应该修的。第三怀疑排除。**

---

## 第四步：换个角度——让朋友帮我试

我让朋友在浏览器里访问 `marlin-phone.github.io`，结果他直接跳转到了 `marlin-phone.me`，一切正常。

而我自己用**无痕窗口**打开，也是正常的。

**问题出在我自己的浏览器上。不是服务器端的问题。**

---

## 第五步：抓出真凶——Service Worker

我打开 Chrome 开发者工具（F12），进入 **Application → Service Workers**，看到了这个：

```
Source: marlin-phone.github.io
Status: activated and is running
```

一个活跃的 **Service Worker** 正运行在 `github.io` 域名下！

它的 `sw.js` 里有一个 **Cache-First 策略**：

```javascript
// 伪代码还原其逻辑
self.addEventListener('fetch', function(event) {
    // 1. 有缓存 → 直接返回缓存（秒开）
    // 2. 同时后台 fetch 新版本
    // 3. 如果 last-modified 不同 → 提醒用户刷新
    event.respondWith(caches.match(event.request).then(function(cached) {
        return cached || fetch(event.request);
    }));
});
```

这就是完整的因果链：

```
很久以前（CNAME 配置之前）
  → 我访问过 github.io
  → Service Worker 注册并缓存了当时的页面（旧版本）
  → 浏览器记住了这个 SW

后来（配置了自定义域名 CNAME）
  → 我的 github.io 触发了 GitHub 301 重定向
  → 但是！Service Worker 拦截了请求
  → 它觉得："有缓存啊，直接给你！"
  → 后台 fetch 拿到的是重定向后的响应（不同域名）
  → last-modified 比较失效，不触发刷新提醒
  → 我永远看到旧版本

而我朋友（全新访客）：
  → 没有 Service Worker 缓存
  → 正常触发 301 重定向
  → 一切正常
```

---

## 为什么会这样？

关键原因有两个：

### 1. Service Worker 的缓存比 HTTP 缓存更「顽固」

HTTP 缓存（`Cache-Control: max-age=600`）会在 10 分钟后过期。但 Service Worker 的缓存**不会自动过期**——它由 `sw.js` 的逻辑控制，如果逻辑不完善，缓存就会一直存活。

### 2. 跨域名打断了更新检测

`sw.js` 的后台更新检测依赖 `last-modified` 头。正常情况（同域名）下，缓存响应和新 fetch 响应来自同一资源，可以比较。但 301 重定向后，fetch 返回的是完全不同的资源（来自 `marlin-phone.me`），比较逻辑失效。

```
【正常情况】同域名
  缓存: github.io 的老页面 (last-modified: May 20)
  fetch: github.io 的新页面 (last-modified: June 6)
  → 不相等 → 提醒刷新 ✅

【异常情况】跨域名重定向
  缓存: github.io 的老页面 (last-modified: May 20)
  fetch: github.io → 301 → me 的页面 (来自不同源)
  → 比较失联 → 不提醒 ❌
```

---

## 解决方案

发现了问题根源，我做了三件事：

### 方案 1：清除 Service Worker（立即生效）

Chrome DevTools → Application → Service Workers → **Unregister**。

清除后刷新，问题立即解决。但这是治标不治本——其他有缓存的访客也会遇到同样的问题。

### 方案 2：添加 JS 强制跳转（治本）

在 `_includes/head.html` 的 `<head>` 最前面（比任何资源加载都早）加入：

```html
<script>
    (function() {
        var correctDomain = 'marlin-phone.me';
        if (window.location.hostname !== correctDomain && 
            window.location.hostname !== 'localhost' && 
            window.location.hostname !== '127.0.0.1') {
            var newPath = window.location.pathname + window.location.search + window.location.hash;
            window.location.replace('https://' + correctDomain + newPath);
        }
    })();
</script>
```

这段代码在页面解析的第一时间运行，**抢在 Service Worker 返回缓存内容之前**，检测域名是否正确。不对就立刻 `replace` 跳走。

保留 `localhost` 和 `127.0.0.1` 是为了不影响本地调试。

### 方案 3：集中化域名配置（可选优化）

在 `_config.yml` 中添加：

```yaml
canonical_domain: "marlin-phone.me"
```

然后 JS 中使用 `{% raw %}{{ site.canonical_domain }}{% endraw %}` 变量引用。以后换域名只需改一处。

---

## 总结

| 现象                   | 根因                                    | 解决方案                           |
| ---------------------- | --------------------------------------- | ---------------------------------- |
| `github.io` 显示旧内容 | Service Worker 缓存了 CNAME 之前的页面  | 清除 SW + 添加 JS 强制跳转         |
| 不提示刷新             | 跨域名重定向打断了 `last-modified` 比较 | JS 跳转在 SW 之前拦截              |
| GitHub 重定向到 HTTP   | Cloudflare 代理导致 GitHub 看不到 HTTPS | Cloudflare「Always Use HTTPS」兜底 |

### 教训

1. **Service Worker 是双刃剑**：它让页面秒开，但缓存策略不完善时会导致「用户永远看不到更新」
2. **域名迁移时，注意清理 SW**：CNAME 配置后，旧的 SW 注册还在，且跨域名的更新检测会失效
3. **JS 跳转比 HTTP 301 更可靠**：301 可以被 SW 拦截，但内联 `<script>` 在 HTML 里，SW 缓存的就是包含跳转逻辑的页面——天然免疫
4. **无痕窗口是最好的排查工具**：服务器端逻辑 vs 浏览器缓存，一测便知

---

> 一个域名、两个面孔的灵异事件，最终竟然是因为我自己浏览器的 Service Worker 在"做好事"。它太尽责了，以至于把旧版本保护得太好 😄