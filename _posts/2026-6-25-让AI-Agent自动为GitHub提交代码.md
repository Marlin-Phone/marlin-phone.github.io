---
layout: post
mathjax: true
title: "让 AI Agent 自动为 GitHub 提交代码"
subtitle: "从 GitHub API 到 Skill 注册，实现一句话自动写博客"
date: 2026-6-25 03:19:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - agent
  - github
  - automation
  - devops
---
## 一、痛点

我有一个个人博客（marlin-phone.me），基于 GitHub Pages + Jekyll 搭建。每次写博客的流程是这样的：

```
打开 IDE → 写 Markdown → 填 front matter → git add → git commit → git push → 等 Actions 部署
```

虽然不复杂，但每次都要走一遍。更重要的是，我的 AI Agent（Friday）经常在跟我对话时生成一些很有价值的内容——**如果它能直接把内容写成博客推送到 GitHub**，我就不用再手动搬运了。

于是我想实现一个目标：

> **我跟 Agent 说一句话，它就能自动写一篇博客，推送到我的 GitHub Pages 仓库，触发自动部署。**

这篇文章就是整个实现过程的技术记录。

## 二、核心架构

整个流程的架构非常简洁：

```
我说"写一篇博客"
    │
    ▼
Agent 生成 Markdown 内容（含 Jekyll front matter）
    │
    ▼
通过 GitHub API 直接写入 _posts/ 目录
    │
    ▼
GitHub Actions 自动构建 → 部署上线
```

这里没有用 `git clone → write → commit → push` 的传统方式，而是直接调用 **GitHub Contents API**。原因很简单：Agent 的工作环境里不一定有 Git 配置，SSH Key 也不方便管理，而 API 只需要一个 Token。

## 三、第一步：获取权限——GitHub Personal Access Token

要让 Agent 往你的仓库写东西，它需要一个凭证。这个凭证就是 GitHub 的 **Personal Access Token (PAT)**。

### 创建步骤

1. 打开 https://github.com/settings/tokens
2. 选择 **Generate new token → Fine-grained token**
3. 设置权限：
   - **Repository access**：Only select repositories → 选你的博客仓库
   - **Permissions**：Contents → **Write**

### 存储方式

Token 存储在 Agent 工作区的 `.github_token` 文件中，只有 Agent 自己能读取：

```bash
# .github_token（注意：这个文件不应该被分享或提交到 Git）
github_pat_11BI2SKHA0Qktnlv6gqtcL_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## 四、第二步：通过 GitHub API 写入文件

GitHub 提供了一个非常方便的 API 来直接创建或更新文件：

```http
PUT /repos/{owner}/{repo}/contents/{path}
```

请求体包含三个关键字段：

```json
{
  "message": "feat: 添加 Agent Workflow 设计范式详解博客",
  "content": "LS0tCmxheW91dDogcG9zdAptYXRoamF4OiB0cnVlCnRpdGxl...(Base64编码的文件内容)",
  "branch": "master"
}
```

对应的 Python 实现非常简洁：

```python
import base64, json, requests

def push_blog_post(token, repo, path, content, commit_msg):
    url = f"https://api.github.com/repos/{repo}/contents/{path}"
    
    data = {
        "message": commit_msg,
        "content": base64.b64encode(content.encode()).decode(),
        "branch": "master"
    }
    
    resp = requests.put(url, json=data, headers={
        "Authorization": f"Bearer {token}"
    })
    
    if resp.status_code == 201:
        print(f"✅ 发布成功: {resp.json()['content']['sha'][:12]}")
    else:
        print(f"❌ 失败: {resp.json()['message']}")
```

这里有几个技术细节需要注意：

**a) Base64 编码**：文件内容必须是 Base64 编码的字符串，不能直接传原文。

**b) 分支指定**：必须指定 `branch` 参数，否则默认使用仓库的默认分支。

**c) 文件路径**：路径相对于仓库根目录，例如 `_posts/2026-6-25-xxx.md`。

## 五、第三步：处理 Jekyll 格式

由于我的博客基于 Jekyll，每篇文章都需要 YAML front matter 头信息。Agent 生成博客时必须包含：

```markdown
---
layout: post
mathjax: true
title: "文章标题"
date: 2026-6-25 03:19:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - agent
  - github
---

## 正文内容
```

文件名也有规范：`_posts/YYYY-M-D-title.md`。例如 `_posts/2026-6-25-Agent-Workflow.md`。

这个格式是 Jekyll 的硬性要求，Agent 必须严格遵守，否则博客解析会出错。

## 六、第四步：Human-in-the-Loop——人工审批

让 Agent 直接推送内容到生产环境是有风险的。如果它生成的内容有问题，或者理解错了我的意思，直接上线就尴尬了。

因此我在流程中加入了**人工审批节点**：

```
Agent 生成博客
    │
    ▼
发送给我审查 ← 新增的人工节点
    │
    ├── 我同意 → 推送上线
    └── 我修改 → Agent 修改后再次审查
```

这个设计遵循了生产环境 Workflow 的核心原则：**LLM 做推理和生成，人类做决策和审批。**

## 七、第五步：注册为 Skill——固化流程

最后一步也是最关键的一步：把整个流程固化，让 Agent 永久记住。

在 Agent 的工作区创建 `skills/blog_writer/SKILL.md`，记录了完整的执行流程：

```
### 执行流程

第零步：获取当前时间（get_current_time）
第一步：生成符合 Jekyll 格式的 Markdown 文件
第二步：发送给 Marlin 审查
第三步：批准后通过 GitHub API 推送
第四步：清理临时文件
```

同时还有一个 `_blog_pusher.py` 推送脚本，封装了 GitHub API 调用。

这样做的意义是：**Agent 重启、会话切换、甚至升级都不会丢失这个能力。** 只要我说"写一篇博客"，它就会读 SKILL.md，按流程执行。

## 八、踩过的坑

### 坑 1：时区问题

第一次测试时，Agent 用了系统时间（可能是 UTC）作为文章日期，比我所在的 Asia/Shanghai 时区晚了 8 小时。**修复方式：强制要求 Agent 在写博客前先调用时间工具获取上海时间。**

### 坑 2：Git 提交规范

Agent 第一次推送时用了 `博客: xxx` 的 commit message，但我项目里用的是 Conventional Commits 风格（`feat: 添加xxx博客` / `fix: 修正xxx`）。**修复方式：在 Skill 中明确指定 commit 规范。**

### 坑 3：Shell 环境差异

Agent 的执行环境是 `/bin/sh`，不是 bash。多行字符串、变量扩展等行为不同。用 shell 写复杂逻辑时经常踩坑。**最终方案：用 Python 写推送脚本，避免 shell 兼容性问题。**

## 九、总结

让 Agent 自动提交 GitHub 的完整技术栈：

| 组件 | 技术选型 |
|------|---------|
| 认证 | GitHub Fine-grained PAT |
| 文件写入 | GitHub Contents API (PUT) |
| 内容格式 | Markdown + Jekyll front matter |
| 部署触发 | GitHub Actions（已有的） |
| 流程编排 | SKILL.md + Python 脚本 |
| 审批机制 | Human-in-the-Loop |
| 版本规范 | Conventional Commits |

整个过程不依赖任何第三方服务，纯 GitHub 生态 + 一个 Agent 就能跑通。

如果你也在用 Agent，想让它能自动提交代码、写博客、提 PR，这套方案可以直接复用。不只是博客——**任何需要 Agent 往 GitHub 写东西的场景**（自动修 Bug 提 PR、自动更新文档、自动生成 Release Notes）都能用同样的模式实现。