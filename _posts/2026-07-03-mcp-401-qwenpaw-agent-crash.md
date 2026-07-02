---
layout: post
mathjax: true
title: "一次 MCP 401 引发的血案：QwenPaw Agent 连环故障排查与修复"
subtitle: "从微信机器人卡死到四个框架级设计缺陷的发现之旅"
date: 2026-07-03 00:52:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - Agent
  - QwenPaw
  - Debug
  - Open Source
  - MCP
---

> ⚠️ **AI 生成标识** · 本文由 AI Agent 协助撰写，人类作者审核发布。

## 一、用户报障：微信机器人"死"了

某天，我给 life_agent（我的微信端 AI 助手）发了条消息，然后——什么都没有发生。不是回复慢，是**完全没有回复**。屏幕上只有一行"正在输入..."，转了五分钟、十分钟，直到我失去耐心关掉了对话框。

作为开发者，第一反应是看日志。日志里赫然躺着一行：

```
HTTP 401 Unauthorized — MCP fetch tool requires OAuth authentication
```

我当时配了一个 ModelScope 的 MCP fetch 工具，但这个工具需要 OAuth 授权。我没在 QwenPaw 的管理面板里完成授权流程，所以每次调用都返回 401。

但问题来了——**一个工具返回 401，应该只是"调用失败"，而不该把整个 agent 弄死吧？**

事情没那么简单。

## 二、追踪：从症状到根因

我拉出完整链路开始排查。正常的消息处理流程是这样的：

```
用户发消息 → start_typing（微信开始显示"正在输入"）
  → 消息入队 → 消费者取消息 → 调用 Agent
    → Agent 推理 → 决定调用工具 → 工具返回结果
      → Agent 继续推理 → 生成回复 → send 到微信
        → stop_typing（微信停止"正在输入"）✅
```

而当 MCP 返回 401 时，链路变成了这样：

```
... → Agent 决定调用工具 → MCP 返回 401
  → Agent._acting() 抛异常（无 except 捕获）
    → 异常向上传播 → 消费者卡死
      → stop_typing 永远不会被调用
        → 微信"正在输入..."永远旋转 💀
```

关键转折点在于 `_acting()` 方法的异常处理。

## 三、第一块多米诺：`_acting()` 的"裸 try"

`_acting()` 是 AgentScope 框架中 Agent 执行工具调用的核心方法。它的结构是这样的：

```python
try:
    tool_res = await self.toolkit.call_tool_function(tool_call)
    async for chunk in tool_res:
        # 处理工具返回的流式结果...
finally:
    await self.memory.add(tool_res_msg)  # 记录到记忆
```

你发现问题了吗？**`try` 块只有 `finally`，没有 `except`。**

这意味着当 `call_tool_function` 抛出任何异常——网络超时、401 未授权、500 服务器错误——这个异常不会被捕获。它会直接向上传播，炸掉 Agent 的整个推理循环。

`finally` 块确实会执行（记录一条半成品的工具结果到记忆），但这救不了 Agent——异常已经飞出去了。

## 四、第二块多米诺：消费者死锁

异常从 `_acting()` 飞出去后，沿着调用链向上：

```
_acting() → _process() → _stream_with_tracker() → _consume_with_tracker()
  → _consume_one_request() → _process_batch() → _consume_queue()
```

`_consume_queue()` 是消息消费者的主循环，它的核心是这样的：

```python
while True:
    payload = await queue.get()
    # ...
    await _process_batch(ch, batch)  # ← 在这里卡死
```

`_process_batch` 调用 Agent 处理消息。当 Agent 因为工具异常而崩溃、但没有把异常正确传回消费者时，消费者就**永远卡在 `await _process_batch` 这一行**。

更糟糕的是，这个消费者对应的是一个 session 队列。一旦它卡死，这个 session 的所有后续消息都被丢弃——用户发再多消息都不会有回应。

## 五、第三块多米诺：typing 指示器永不停止

微信的"正在输入..."是通过一个后台协程实现的：

```python
async def refresh_typing():
    while not stop_event.is_set():
        await client.sendtyping(user_id, ticket, status=1)
        await asyncio.wait_for(stop_event.wait(), timeout=5.0)
```

这个协程每 5 秒刷新一次 typing 状态，直到 `stop_event` 被设置。而 `stop_event` 是在正常完成流程中（`_on_consume_finish` 或 `_on_consume_error`）才被设置的。

当消费者卡死，这些完成流程永远不会执行，`stop_event` 永远不会被设置，`refresh_typing` 就永远跑下去。

从用户视角看，就是微信对话框里那个"正在输入..."永远在跳动。用户体验极差。

## 六、四个设计缺陷全景

| # | 位置 | 问题 | 后果 |
|---|------|------|------|
| 1 | `_react_agent.py` `_acting()` | `try` 无 `except`，只有 `finally` | 工具异常直接向上传播，无恢复逻辑 |
| 2 | `unified_queue_manager.py` 消费者 | 无执行超时 | 卡住就永远卡住 |
| 3 | `base.py` session 队列 | 卡死任务阻塞整个队列 | 后续消息全部丢弃 |
| 4 | WeChat typing 指示器 | `start_typing` 后无超时/错误恢复 | typing 永不消失 |

这四个缺陷中的任何一个单独存在，可能都不是致命问题。但当它们串在一起时，就构成了一条完美的"故障链"——一个 401 就能让整个系统瘫痪。

## 七、修复：三道防线

### 修复 1：工具异常兜底（AgentScope）

在 `_acting()` 的 `try` 和 `finally` 之间，增加 `except Exception`：

```python
except Exception as e:
    error_msg = f"Tool call failed: {e}"
    tool_res_msg.content[0]["output"] = error_msg
    await self.print(tool_res_msg, True)
    return None
```

这样，工具调用失败后，Agent 会收到一条错误消息（而不是崩溃），它可以基于错误信息调整策略、尝试其他工具、或者如实告诉用户"我无法完成这个操作"。

### 修复 2：消费者超时保护（QwenPaw）

在 `manager.py` 中用 `asyncio.wait_for` 包裹 `_process_batch`：

```python
try:
    await asyncio.wait_for(
        _process_batch(ch, batch),
        timeout=300.0,  # 5 分钟
    )
except asyncio.TimeoutError:
    logger.error("Process batch timed out")
    # 尝试停止 typing 指示器
    ch._stop_typing_for_user(user_id)
    continue  # 继续处理下一条消息
```

### 修复 3：typing 指示器安全网（QwenPaw）

在 `refresh_typing()` 中添加最大持续时间：

```python
started_at = time.monotonic()
MAX_TYPING_SECONDS = 300  # 5 分钟
while not stop_event.is_set():
    if time.monotonic() - started_at > MAX_TYPING_SECONDS:
        logger.warning("typing max duration exceeded, auto-stopping")
        break
    # ...
```

即使 Agent 异常挂起，typing 也不会永远旋转。

## 八、教训与开源贡献

这次排障让我重新认识了"防御性设计"的重要性：

1. **异常处理要兜底**。`try` 块不能只有 `finally`。永远假设外部调用可能失败，给 Agent 一个优雅降级的机会。
2. **超时是必需品，不是可选项**。任何异步等待都应该有超时。没有超时的代码就像没有刹车的车。
3. **指示器要有自毁机制**。typing、loading、progress bar——任何"等待中"的 UI 元素都必须有最大持续时间，这是对用户的尊重。
4. **故障链思维**。单点缺陷可能无害，但多个缺陷串联就是灾难。排障时不能只看最近的错误，要看整条链。

我已经把这三个修复提交给了上游仓库：

- **AgentScope** [PR #1986](https://github.com/agentscope-ai/agentscope/pull/1986)：`fix(agent): catch tool call exceptions in _acting() to prevent crash`
- **QwenPaw** [PR #5749](https://github.com/agentscope-ai/QwenPaw/pull/5749)：`fix(channel): add consumer timeout and typing auto-stop to prevent agent hang`

希望这些修复能让使用 QwenPaw 的开发者少踩一个坑。

---

*如果你也在用 QwenPaw 遇到类似"微信机器人不回复"的问题，检查一下你的 MCP 工具是否需要 OAuth 授权。先到管理面板完成授权，再配工具。这个坑我已经替你们踩过了。*