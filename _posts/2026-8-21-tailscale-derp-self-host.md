---
layout: post
mathjax: true
title: "Tailscale 打洞失败？自建 DERP 中继 10 分钟搞定"
subtitle: "从 327ms 降到 18ms，告别绕地球半圈"
date: 2026-8-21 03:08:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - Tailscale
  - DERP
  - 网络
  - 内网穿透
---

> ⚠️ **AI 生成标识** · 本文由 AI Agent 协助撰写，人类作者审核发布。

## 背景

我在京东云有一台服务器（天津），家里有一台内网服务器（西安），通过 Tailscale 组成虚拟局域网。部署 Docker 容器、SSH 连接都很方便。

但最近发现一个问题：**通过 Tailscale 访问家里服务器非常卡**，Ping 延迟高达 **327ms**，而且有抖动（831ms、754ms）。

## 排查过程

### 第一步：确认 Tailscale 是否直连

```
$ tailscale netcheck

Nearest DERP: San Francisco
DERP latency:
	- sfo: 144.3ms (San Francisco)
	- lax: 155ms   (Los Angeles)
	- ...
```

**发现问题了！** Tailscale 返回的最近 DERP 节点是美国旧金山，延迟 144ms。

### 第二步：确认连接方式

```
$ tailscale ping 100.72.204.6

pong from marlin-server (100.72.204.6) via DERP(sfo) in 327ms
```

确认走的是 `DERP(sfo)` —— **连接没有建立 P2P 直连，而是通过美国旧金山的中继服务器转发**！

### 问题根源

**京东云的数据中心 NAT 和西安电信的家庭 NAT 不兼容**，Tailscale 的 UDP 打洞（STUN）失败，只能降级走 DERP 中继。

数据包路径变成了：

```
你（西安）→ 美国旧金山 DERP → 京东云（天津）
```

绕了半个地球，延迟自然爆炸。

## 解决方案：自建 DERP 中继

Tailscale 官方的 DERP 服务器是公用的，默认中继节点分布在全球。**我可以自己搭一个 DERP 中继节点**放在京东云（天津），这样中继路径就变成了：

```
你（西安）→ 天津 DERP（京东云）→ 天津服务器
```

延迟从 327ms 直接降到 30ms 以内！

### 搭建步骤

#### 1. 拉取 DERP 镜像

```bash
docker pull fredliang/derper:latest
```

#### 2. 启动 DERP 容器

```bash
docker run -d --name derper \
  --restart unless-stopped \
  -p 443:443 \
  -p 3478:3478/udp \
  -v /root/derper-data:/app/certs \
  -e DERP_DOMAIN=你的服务器IP \
  -e DERP_CERT_MODE=manual \
  -e DERP_ADDR=:443 \
  -e DERP_STUN=true \
  -e DERP_STUN_PORT=3478 \
  fredliang/derper:latest
```

- `443`：HTTPS 控制通道
- `3478/udp`：STUN 打洞服务
- 自签证书，不需要域名

启动后日志会输出一个关键信息：

```
Using self-signed certificate for IP address "xxx.xxx.xxx.xxx".
Configure it in DERPMap using:
  {"Name":"custom","RegionID":900,"HostName":"xxx.xxx.xxx.xxx","CertName":"sha256-raw:xxxxxxxx"}
```

**这个 `CertName` 一定要记下来**，后面配置要用。

#### 3. 配置 Tailscale ACL

登录 [Tailscale 管理后台](https://login.tailscale.com/admin/acls)，在 ACL JSON 中添加：

```json
{
  "derpMap": {
    "OmitDefaultRegions": false,
    "Regions": {
      "900": {
        "RegionID": 900,
        "RegionCode": "my-derp",
        "RegionName": "自定义中继",
        "Nodes": [{
          "Name": "1",
          "RegionID": 900,
          "CertName": "sha256-raw:上面日志里的值",
          "HostName": "你的服务器IP",
          "STUNPort": 3478
        }]
      }
    }
  }
}
```

#### 4. 重启 Tailscale 生效

```bash
sudo tailscale down && sudo tailscale up
```

## 效果

### 延迟对比

| 指标 | 之前（DERP sfo） | 现在（DERP 自建） |
|:-----|:-----------------|:------------------|
| 延迟 | 327ms | **18ms** |
| 中继路径 | 天津→旧金山→西安 | 天津→天津→西安 |
| 访问服务耗时 | — | **0.04s** |

### 带宽测试

```
$ tailscale ping 100.72.204.6

pong from marlin-server (100.72.204.6) via DERP(my-derp) in 18ms
pong from marlin-server (100.72.204.6) via DERP(my-derp) in 18ms
pong from marlin-server (100.72.204.6) via DERP(my-derp) in 18ms
```

**延迟降低 94%，效果立竿见影！**

## 避坑指南

1. **`CertName` 必须正确**：自签证书模式下，Tailscale 用 `CertName`（证书指纹）验证身份，而不是 `DERPPort`。填错会导致节点无法连接。

2. **`OmitDefaultRegions` 建议设 `false`**：这样官方 DERP 节点仍然可用作兜底，自定义节点不可用时自动 fallback。

3. **防火墙放行**：别忘了在安全组/防火墙放行 `443/tcp` 和 `3478/udp`。

4. **带宽共享**：DERP 中继走的是你服务器的带宽，所有流量共享。服务器带宽小的话要考虑清楚（比如我京东云只有 5Mbps）。

5. **手机/电脑端也要重启**：Tailscale App 本地缓存了 DERP map，需要在客户端重启才能使用新的中继节点。

## 延伸思考

Tailscale 的 DERP 机制设计很有意思：**先尝试 UDP 打洞（P2P），失败才降级走 DERP 中继**。这保证了可靠性也兼顾了性能。

对我个人来说，**自建 DERP 比用 FRP 更省心**——FRP 需要为每个服务手动配置端口转发，而 Tailscale + 自建 DERP 只是把"打洞失败"的场景修复了，所有 Tailscale 服务自动生效，不需要改任何配置。

如果你也有 Tailscale 跨地区访问卡顿的问题，不妨试试自建 DERP。