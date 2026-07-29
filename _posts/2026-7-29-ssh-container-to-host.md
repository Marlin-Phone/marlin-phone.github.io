---
layout: post
mathjax: true
title: "让容器里的 AI Agent 通过 SSH 访问宿主机：原理与实操"
subtitle: "Docker 多网桥、SSH 密钥认证与 sudo 权限升级"
date: 2026-7-29 20:24:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - Docker
  - SSH
  - Linux
  - Agent
  - 运维
---

> ⚠️ **AI 生成标识** · 本文由 AI Agent 协助撰写，人类作者审核发布。

## 前言

想象这样一个场景：你有一个 AI Agent 跑在 Docker 容器里，它需要访问宿主机上的资源——查日志、看进程、管理文件、甚至部署服务。但容器默认是隔离的，怎么让它"走出"容器，触达宿主机？

最近我在部署自己的 AI Agent 时遇到了这个问题，最终通过 SSH 密钥认证 + sudo 权限配置解决了。这篇博客聊聊背后的原理和具体操作。

## 一、Docker 网络：容器与宿主机是邻居

### 1.1 默认的 Docker 网络

当我们启动一个 Docker 容器时（不指定 `--network`），它默认连接到 `docker0` 网桥：

```
宿主机
┌────────────────────────────────────┐
│  eth0 (公网 IP)                     │
│                                      │
│  docker0 网桥 (172.17.0.1/16)       │
│       │                              │
│       ├── 容器 A (172.17.0.2)        │
│       ├── 容器 B (172.17.0.3)        │
│       └── ...                        │
│                                      │
│  SSH 服务 (sshd) 监听 :22           │
└────────────────────────────────────┘
```

**关键认知：** 宿主机在 `docker0` 网桥上的 IP 永远是 `172.17.0.1`（网桥网关）。容器在同一个二层网络中，可以直接访问宿主机——不需要经过公网，没有 NAT 穿透问题。

### 1.2 实际场景：多网桥接入

但实际情况可能更复杂。在我部署的 QwenPaw 容器中，容器同时接入了多个 Docker 网桥：

```bash
# 容器内查看网络接口
$ hostname -I
172.16.0.10 172.17.0.1 172.18.0.1 172.19.0.1

# 路由表
$ cat /proc/net/route
eth0    172.16.0.0/16    直连     # 自定义网桥
docker0 172.17.0.0/16    直连     # 默认 Docker 网桥
br-...  172.18.0.0/16    直连     # 其他自定义网桥
br-...  172.19.0.0/16    直连     # 其他自定义网桥
```

容器的实际网络拓扑：

```
宿主机
┌─────────────────────────────────────────────────┐
│  docker0 (172.17.0.1/16)   自定义网桥 (172.16.0.1/28)  │
│       │                            │                  │
│       └──────────┬─────────────────┘                  │
│                  │                                     │
│            QwenPaw 容器                                │
│          ┌────────────────┐                            │
│          │ eth0: 172.16.0.10  │                        │
│          │ docker0: 172.17.0.x │                        │
│          └────────────────┘                            │
│                                                         │
│  SSH 服务 (sshd) 监听 :22                              │
└─────────────────────────────────────────────────────────┘
```

这意味着容器同时在多个二层网络中，**通过 `docker0` 网桥的 `172.17.0.1` 直连宿主机**，这是 SSH 访问的基础。

## 二、SSH 密钥认证：比密码更安全

### 2.1 为什么不用密码？

你可能会想：既然宿主机有 SSH 服务，用户名密码登录不就行了？

有两个问题：

1. **密码在网络中传输**（即使是加密的 SSH 协议，密码认证也需要交互式输入，不适合自动化场景）
2. **Agent 需要无交互登录**——它不能弹出一个"请输入密码"的对话框

### 2.2 密钥认证的工作原理

SSH 密钥认证使用**非对称加密**（ED25519 或 RSA），流程如下：

```
容器（客户端）                    宿主机（服务端）
     │                                 │
     │  1. 发起 SSH 连接请求            │
     │─────────────────────────────────>│
     │                                 │
     │  2. 查 ~/.ssh/authorized_keys    │
     │  3. 生成随机数 challenge         │
     │<─────────────────────────────────│
     │                                 │
     │  4. 用私钥签名 challenge         │
     │─────────────────────────────────>│
     │                                 │
     │  5. 用公钥验证签名               │
     │  6. 验证通过 → 登录成功!         │
     │<─────────────────────────────────│
```

**安全优势：**
- **私钥永不出客户端**——只在本地签名，不传输
- **公钥即使泄露**——没有私钥也登不上
- **挑战-应答机制**——防止重放攻击
- 比密码更安全，同时支持**无交互自动化**

### 2.3 实操：生成密钥对

容器内生成 ED25519 密钥对（比 RSA 更短、更快、更安全）：

```bash
ssh-keygen -t ed25519 -f ~/.ssh/host_key -N ""
```

得到两个文件：
- `~/.ssh/host_key` —— **私钥**（权限 600，绝不能泄露）
- `~/.ssh/host_key.pub` —— **公钥**（可以公开）

### 2.4 部署公钥到宿主机

把公钥内容追加到宿主机 `ai` 用户的 `~/.ssh/authorized_keys`：

```bash
# 宿主机上执行
echo 'ssh-ed25519 AAAA... agent-friday@container' >> ~ai/.ssh/authorized_keys
chmod 600 ~ai/.ssh/authorized_keys
chmod 700 ~ai/.ssh
chown -R ai:ai ~ai/.ssh
```

**权限很重要：** `authorized_keys` 必须是 `600`，`.ssh` 目录必须是 `700`，否则 sshd 会拒绝使用。

## 三、SSH Config：从繁琐到优雅

### 3.1 没有配置的痛苦

没有配置时，每次连接要敲：

```bash
ssh -i ~/.ssh/host_key -o StrictHostKeyChecking=no ai@172.17.0.1
```

参数又多又长，还容易敲错。

### 3.2 配置 SSH Config

在 `~/.ssh/config` 中添加：

```
Host host
    HostName 172.17.0.1
    User ai
    IdentityFile ~/.ssh/host_key
    StrictHostKeyChecking no
```

就这样，以后在容器内只需要：

```bash
ssh host          # 登录
ssh host "命令"   # 远程执行
ssh host < script # 远程执行脚本
```

简洁、优雅、无交互。

## 四、持久化：容器重建怎么办？

### 4.1 Docker 容器的文件系统

Docker 容器的文件系统分为两类：

| 路径 | 持久性 | 说明 |
|------|--------|------|
| `/tmp/` | ❌ 销毁 | 随容器销毁 |
| `/root/.ssh/` | ❌ overlay 层 | 重建丢失 |
| 挂载卷（如 `/app/working/`） | ✅ 持久化 | 宿主机磁盘，永久保留 |

### 4.2 持久化方案

把 SSH 密钥和配置放在挂载卷中，容器启动时自动恢复：

```bash
# 备份位置：/app/working/.ssh_host/
# 包含：host_key, host_key.pub, config, restore_ssh.sh
# 容器重建后执行：
bash /app/working/.ssh_host/restore_ssh.sh
```

甚至可以更进一步——Agent 每次需要 SSH 时自动检测连接是否可用，失效则自动恢复，对用户完全透明。

## 五、权限升级：从普通用户到 sudo

### 5.1 普通用户的限制

SSH 登录到宿主机后，`ai` 用户默认是**受限普通用户**：

```bash
$ groups ai
ai users

# 不能操作 Docker
$ docker ps
permission denied while trying to connect to the Docker socket

# 不能读系统日志
$ cat /var/log/syslog
Permission denied

# 不能重启服务
$ systemctl restart xxx
Failed to restart xxx: Access denied
```

### 5.2 加入 sudo 组

要让 Agent 真正能干"运维活"，需要给 `ai` 用户赋予 sudo 权限：

```bash
# root 执行
usermod -aG sudo ai
```

但默认的 sudo 配置需要密码，Agent 的非交互式 SSH 无法输入密码。所以需要配置 NOPASSWD：

```bash
# root 执行
echo "ai ALL=(ALL) NOPASSWD:ALL" | tee /etc/sudoers.d/ai
```

验证：

```bash
$ sudo -n whoami
root
```

**`-n` 参数表示非交互模式**，如果 sudo 需要密码但没提供，会直接报错而不是挂起等待。配置 NOPASSWD 后，Agent 可以放心使用 `sudo` 执行任意命令。

### 5.3 升级后的能力对比

| 操作 | 普通用户 | 有 sudo 权限 |
|------|---------|-------------|
| 查看系统资源 | ✅ `free -h` | ✅ 不变 |
| 操作 Docker | ❌ | ✅ `sudo docker ps/logs/stats` |
| 读系统日志 | ❌ | ✅ `sudo tail -f /var/log/syslog` |
| 重启服务 | ❌ | ✅ `sudo systemctl restart xxx` |
| 查看任意文件 | ❌ | ✅ `sudo cat /etc/sudoers` |
| 管理容器 | ❌ | ✅ `sudo docker rm/stop/exec` |

有了 sudo，Agent 的能力边界从"查看"扩展到了"操控"。

## 六、一些思考

### 6.1 为什么选择 SSH 而不是其他方式？

其实还有几种方案可以实现容器→宿主机通信：

| 方案 | 优点 | 缺点 |
|------|------|------|
| **SSH** | 标准、安全、无侵入 | 需要额外配置 |
| Docker Socket 挂载 | 可以直接管理 Docker | 权限过大，不安全 |
| HTTP API | 需要额外开发 | 需要部署服务端 |
| 共享 Volume | 简单直接 | 只能读写文件，不能执行命令 |

SSH 胜在**标准化**——几乎任何 Linux 宿主机都自带 sshd，不需要额外安装任何软件，而且支持执行任意命令、传输文件、端口转发，非常灵活。

### 6.2 Agent 的"延伸"之路

从 Agent 的角度看，SSH 到宿主机意味着它的能力边界从容器扩展到了整个宿主机，再配合 sudo 权限，可以做到：

```
容器内 Agent
    │
    ├── SSH → 宿主机 shell（任意命令）
    │     ├── sudo docker → 管理容器
    │     ├── sudo systemctl → 管理服务
    │     ├── sudo tail/vim → 读/写系统文件
    │     └── sudo apt → 安装软件
    │
    ├── SCP → 宿主机文件系统（传输文件）
    │
    └── 端口转发 → 宿主机网络（访问内部服务）
```

这是 Agent 与基础设施融合的关键一步——从"容器里的一个对话窗口"到"能真正干活的运维助手"。

## 总结

整个过程的核心思路是：

1. **容器和宿主机在同一个网络里**，这是通信的基础
2. **SSH 密钥认证**比密码更安全、更适合自动化
3. **SSH Config** 把繁琐的命令简化为一句 `ssh host`
4. **注意持久化**，容器重建后需要恢复配置
5. **sudo 权限**让 Agent 从"只读"升级到"可操控"，真正具备运维能力

有了这套配置，Docker 容器里的 AI Agent 就能像本地管理员一样访问和操作宿主机了。

---

*如果你也有类似的 Agent 部署需求，或者有其他有趣的"容器跨界"方案，欢迎交流讨论！*