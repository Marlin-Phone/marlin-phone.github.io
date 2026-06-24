---
layout: post
mathjax: true
title: "Agent Workflow 设计范式详解"
subtitle: "从 ReAct 到 Orchestrator，生产环境落地实践"
date: 2026-6-25 01:30:00
author: "Marlin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - agent
  - workflow
  - llm
---
## 一、什么是 Agent Workflow

Agent Workflow 是指将 LLM 调用、Tool 调用与决策逻辑编排成一个可执行流程的框架。它比单个 Tool 调用高一个抽象层次，决定了"什么时候调什么 Tool，调完怎么处理"。

## 二、六大设计范式

### 1. ReAct（Reasoning + Acting）
核心思想：让 LLM 交替进行推理思考和行动调用，每一步的思考结果指导下一步行动。

### 2. Plan-and-Solve
核心思想：先制定完整计划，再逐步执行，执行过程中可调整。

### 3. Multi-Agent
核心思想：多个 Agent 各司其职，通过消息传递协作完成复杂任务。

### 4. Memory-Augmented
核心思想：给 Agent 配备短期和长期记忆系统，在多次对话中保持上下文。

### 5. Tool-Augmented
核心思想：通过注册大量工具扩展 Agent 的能力边界。

### 6. Reflection
核心思想：Agent 在执行后自我审查、自我修正，提高输出质量。

## 三、生产环境实践

在生产环境中，Workflow 很少使用声明式框架，而是通过队列、线程池和显式状态管理实现。以卫星遥测诊断系统为例：

```
MQ/Kafka -> RecvProcess -> Judge -> RealtimeResultProcess -> FaultProcess -> 告警
```

每一步之间用队列解耦，保证处理顺序和时序隔离。每个阶段有独立的线程池，互不影响。

## 四、GitHub Copilot Workspace 案例

Copilot Workspace 是一个完整的 Plan-and-Solve 落地案例：
1. Spec 生成：LLM 理解 Issue，生成规格说明
2. 计划生成：LLM 分析代码，生成修改计划
3. 代码执行：Tool 逐文件生成修改
4. 自检修复：LLM 审查代码并自动修复
5. PR 生成：输出完整 Pull Request

核心规律：复杂度的 80% 不在 AI 推理，而在如何与现有系统集成、如何让人类参与、如何容错和回退。