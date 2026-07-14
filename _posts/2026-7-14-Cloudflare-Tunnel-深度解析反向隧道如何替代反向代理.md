---
layout: post
mathjax: true
title: "Cloudflare Tunnel 深度解析：反向隧道如何替代反向代理"
subtitle: "从原理到架构，理解为什么它能让你关掉防火墙"
date: 2026-7-14 21:20:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - Cloudflare
  - 网络
  - HTTPS
  - 架构
---

> ⚠️ **AI 生成标识** · 本文由 AI Agent 协助撰写，人类作者审核发布。

## 传统方案：反向代理

把内网服务暴露到公网，最经典的做法是反向代理。架构长这样：

```
用户 → HTTPS → 公网IP:443 → Nginx/Caddy → localhost:3000（你的服务）
```

这需要做四件事：

1. **买域名，DNS 解析到服务器公网 IP**
2. **装 Nginx 或 Caddy**，写反向代理配置
3. **申请 SSL 证书**（Let's Encrypt / 付费证书），配自动续期
4. **开防火墙 80/443 端口**，让外部流量进来

每一步都有坑。证书过期忘了续、Nginx 配置写错导致 502、防火墙规则太松被扫——这些都是运维的日常痛点。

核心问题在于：**流量是从外面主动打进来的**，所以服务器必须开一个入口等着。

## Cloudflare Tunnel 的思路：反向隧道

Cloudflare Tunnel 换了一个方向。它不在服务器上开入口，而是让服务器**主动向 Cloudflare 建立一个出站连接**：

```
你的服务器 ──出站连接──→ Cloudflare 边缘节点
                              ↑
                            用户请求
```

这个出站连接一旦建立，Cloudflare 就把用户请求通过它"倒灌"回你的服务器。因为是出站连接，所以：

- **不需要开任何入站防火墙端口**
- **不需要公网 IP 都能用**（只要服务器能访问外网）
- **Cloudflare 天然做了一层 DDoS 防护**

这其实就是把"反向代理"翻了个面——不再是外部请求打进内网，而是内网主动把管道伸出去，让请求从管道流回来。所以叫"反向隧道"。

## 协议层面发生了什么

cloudflared 启动后，会通过 QUIC（基于 UDP）和 Cloudflare 边缘节点建立一条长连接。QUIC 的选择很关键：

| 特性 | 为什么重要 |
|------|-----------|
| 基于 UDP | 不需要建立 TCP 连接，NAT 穿透更友好 |
| 0-RTT 握手 | 断线重连几乎无感知 |
| 多路复用 | 一条连接承载多个 HTTP 请求，不出现队头阻塞 |

Tunnel 建立后，Cloudflare 边缘节点就拿到了你配置的路由规则——"`api.你的域名` → `localhost:3000`"。之后任何打到这个域名的 HTTPS 请求，Cloudflare 解密后通过 QUIC 隧道转发到 cloudflared，cloudflared 再转给本地服务。响应沿原路返回。

**SSL 终止点在 Cloudflare 边缘，而不是你服务器上。** 这意味着：

- Cloudflare 帮你处理 TLS 握手
- 证书由 Cloudflare 自动签发和续期（用的是他们自家的 Origin CA 或通用证书）
- 你的服务器不需要接触任何私钥

但这也意味着 **Cloudflare 能看到明文流量**。这是所有 CDN 代理的固有特性——信任模型上，Cloudflare 是一个受信任的中间人。

## 隧道 vs 反向代理：架构对比

| 维度 | Nginx/Caddy 反代 | Cloudflare Tunnel |
|------|-----------------|-------------------|
| 流量方向 | 外部 → 服务器（入站） | 服务器 → Cloudflare（出站） |
| 防火墙 | 必须开 80/443 | 不需要开任何端口 |
| SSL 证书 | 自己申请 + 续期 | Cloudflare 自动管理 |
| 延迟 | 直连，~1ms | 多一跳，+30~50ms |
| DDoS 防护 | 自己扛 | Cloudflare 边缘扛 |
| 信任模型 | 自己持有私钥 | Cloudflare 是中间人 |
| 大文件 | 无限制 | 免费版 100MB 限制 |
| 内部服务调用 | 本地直连，低延迟 | 绕公网，不合适 |

关键区别不是"谁更好"，而是**适用场景不同**：

- **对外暴露 API / Web 服务**：Tunnel 省心，安全，零维护
- **微服务间内部通信**：Nginx 本地直连，延迟低，不依赖外部
- **复杂路由 / 限流 / 缓存**：Nginx 有丰富的模块生态

两者不是互斥的——你可以用 Tunnel 把流量从公网接进来，然后在内部用 Nginx 做服务间路由。

## 为什么 DNS 会成为一个坑

Tunnel 创建后，Cloudflare 需要在 DNS 里加一条 CNAME 记录，指向 `<tunnel-id>.cfargotunnel.com`。

更隐蔽的问题是 DNS 缓存。Cloudflare 的权威 DNS（`1.1.1.1`）几秒就生效，但你服务器用的可能是云厂商内网 DNS（比如 `169.254.x.x`），它的缓存策略可能滞后几分钟甚至更久。

现象是：用 `curl --dns-servers 1.1.1.1` 能通，直接 `curl` 不通。解法是把系统 DNS 换成 `1.1.1.1`，或者干脆等。

## 和其他内网穿透方案比

| 方案 | 原理 | 优点 | 缺点 |
|------|------|------|------|
| Cloudflare Tunnel | QUIC 反向隧道 | 免费、自动 HTTPS、自带 CDN | 依赖 Cloudflare、有文件大小限制 |
| frp | 自建中转服务器 | 完全可控、无限制 | 需要自己有公网服务器做中转 |
| ngrok | 商业隧道服务 | 开箱即用 | 免费版有限制、自定义域名收费 |
| Tailscale Funnel | WireGuard + HTTPS | P2P 优先、延迟低 | 需要客户端、生态较小 |

选 Cloudflare Tunnel 的核心原因通常是：**已经有域名在 Cloudflare 上托管 DNS**，那么 Tunnel 就是零额外成本的方案。

## 总结

Cloudflare Tunnel 不是新概念——SSH 反向隧道（`ssh -R`）几十年前就有了。但 Cloudflare 把它和 CDN、SSL 证书管理、DDoS 防护打包在一起，让"暴露内网服务"这件事从"配 Nginx + certbot + 防火墙"退化成了"一条 Docker 命令"。

这是架构上的减法：减去的是你的运维负担，加上的是 Cloudflare 作为中间人的信任依赖。对于个人开发和小团队来说，这个取舍完全值得。

---

> 参考：[Cloudflare Tunnel 官方文档](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)