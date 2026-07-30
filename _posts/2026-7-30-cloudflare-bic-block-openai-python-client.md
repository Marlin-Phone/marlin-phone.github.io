---
layout: post
mathjax: true
title: "Cloudflare 拦截了你的 API？记一次 OpenAI Python 客户端请求头引发的 403 排查"
subtitle: "从 curl 能通到 Python 不通，一步步找到真凶"
date: 2026-7-30 19:34:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - Cloudflare
  - OpenAI
  - Debug
  - API
  - Python
---

> ⚠️ **AI 生成标识** · 本文由 AI Agent 协助撰写，人类作者审核发布。

## 引子

我自建了一个 API 代理（基于 New API），用 Cloudflare Tunnel 暴露到公网，地址是 `https://api.marlin-phone.me/v1`。自己用 curl 测试一切正常：

```bash
$ curl https://api.marlin-phone.me/v1/models
{"data":[...]}  # ✅ 200
```

但朋友配置后，死活获取不到模型。用的是 QwenPaw（跟我一样的 Agent 框架），返回 403。

## 第一反应：权限问题？

先查了 New API 的数据库，确认了：

- 自用模式（SelfUseMode）已开启 → 只有我的 token 能调
- 但朋友用我的 token 也不行

到服务器上 curl 测试：

```bash
$ curl https://api.marlin-phone.me/v1/chat/completions \
  -H "Authorization: Bearer my_token" \
  -d '{"model":"deepseek-ai/DeepSeek-V4-Flash","messages":[{"role":"user","content":"hi"}]}'
# ✅ 200 正常
```

API 本身没问题。那问题在哪？

## 在 QwenPaw 里模拟朋友配置

我在 QwenPaw 里新建了一个 provider，指向 `https://api.marlin-phone.me/v1/`，用同样的 token，创建了一个测试智能体。然后让两个智能体对话：

```
我 → 测试智能体: "回复我：测试通过 ✅"
测试智能体: (沉默)... 返回空
```

查日志发现：

```
Provider 'friend_api' not found.
```

这是一个 provider 加载的 bug，重启后解决了。但接下来才是真正的坑：

```
openai.PermissionDeniedError: Your request was blocked.
```

## 锁定真凶：请求头

我写了段脚本，拦截 OpenAI Python 客户端发出的实际请求：

```python
class DebugClient(httpx.AsyncClient):
    async def send(self, request, **kwargs):
        print(f"Headers: {request.headers}")
        return await super().send(request, **kwargs)

client = AsyncOpenAI(
    base_url="https://api.marlin-phone.me/v1/",
    api_key="my_token",
    http_client=DebugClient(proxy="http://127.0.0.1:7890"),
)
```

输出让我眼前一亮：

```
Header [user-agent]: AsyncOpenAI/Python 2.33.0
Header [x-stainless-lang]: python
Header [x-stainless-package-version]: 2.33.0
Header [x-stainless-os]: Linux
Header [x-stainless-arch]: x64
Header [x-stainless-runtime]: CPython
Header [x-stainless-runtime-version]: 3.11.2
```

**Stainless** 是 OpenAI 用来生成 Python SDK 的工具链。这些 `X-Stainless-*` 头是 OpenAI Python 客户端自动加的遥测信息，用来统计 SDK 版本、操作系统、Python 实现等。

我对比测试了一下：

| User-Agent / 请求头 | 结果 |
|---|---|
| `curl/7.88.1`（无 Stainless 头） | ✅ 200 |
| `AsyncOpenAI/Python 2.33.0`（有 Stainless 头） | ❌ 403 |
| 纯 httpx 请求（无 Stainless 头） | ✅ 200 |

**Cloudflare 识别到 `X-Stainless-*` 这些头，断定这是自动化脚本，直接 403 拦截了。**

## 尝试 WAF 规则

我尝试通过 Cloudflare API 创建 WAF 跳过规则：

```json
{
  "action": "skip",
  "action_parameters": {
    "phases": ["http_request_firewall_managed"]
  }
}
```

规则创建成功，但 `skip` 动作在免费计划上不生效。我又试了 `products: ["bic", "uaBlock", "waf"]` 等参数，都不行。

## 最终答案：Browser Integrity Check

在 Cloudflare 的安全设置里，有一个叫 **Browser Integrity Check（浏览器完整性检查）** 的功能。它会检查请求头是否"像"一个真实浏览器发出的。`X-Stainless-*` 头显然不像是浏览器会发的，于是被拦截了。

关掉它之后：

```bash
$ curl -H "User-Agent: AsyncOpenAI/Python 2.33.0" ...
# ✅ 200 正常！
```

**根因就是 Browser Integrity Check。**

## 两种解决方案

### 方案 A：关掉 BIC（推荐给 API 域名）

去 Cloudflare Dashboard → Security → Settings → 关闭 **Browser Integrity Check**。

对 API 子域名来说是安全的，因为 API 本身有 token 认证。朋友直接配置 URL 和 token 就能用，不需要额外设置。

### 方案 B：保持 BIC 开启，覆盖请求头

如果不想关 BIC，可以在 QwenPaw 的 provider 配置里加 `custom_headers`：

```json
{
  "custom_headers": {
    "User-Agent": "curl/7.88.1",
    "X-Stainless-Lang": "",
    "X-Stainless-Package-Version": "",
    "X-Stainless-OS": "",
    "X-Stainless-Arch": "",
    "X-Stainless-Runtime": "",
    "X-Stainless-Runtime-Version": ""
  }
}
```

这样 `X-Stainless-*` 头被覆盖成空字符串，Cloudflare 就认不出来了。

## 总结

这次排查从"curl 能通，Python 不通"的小问题出发，一步步深挖到 Cloudflare 的 Browser Integrity Check 机制。几个关键教训：

1. **curl 测试通过不代表所有客户端都能通过**——不同客户端有不同的请求头
2. **拦截请求头看真实内容**——比瞎猜高效得多
3. **Cloudflare 的免费计划 WAF 跳过规则不生效**——别在这上面浪费时间
4. **API 域名关掉 BIC 通常没问题**——API 本身有认证，不需要浏览器完整性检查

如果你也遇到类似问题，先检查一下 Browser Integrity Check 的状态吧。