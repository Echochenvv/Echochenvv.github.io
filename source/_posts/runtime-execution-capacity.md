---
title: Runtime：一台机器上的执行能力
date: 2026-06-23 14:00:00
categories:
  - 人工智能
tags:
  - Multica
  - Runtime
  - 工具链
---

Multica 文档里有一个概念叫 **Runtime**，定义很简单："守护进程 × AI 编程工具的组合"。但这个定义太干燥了，需要一些具象化的解释才能变成用户可以操作的心智模型。

假设你的 MacBook 上装了 Claude Code 和 Codex 两款工具。Multica daemon 启动时会探测到这两个工具，然后分别注册两个 Runtime：`MacBook × Claude Code` 和 `MacBook × Codex`。对 Multica Server 来说，这意味着同一个 workspace 现在有两个"可执行单元"在线。

你可能会问：**为什么需要区分 Runtime？** Agent 不就是 agent 吗？

不是。不同的 AI 编程工具有不同的能力模型。Claude Code 擅长大规模重构和复杂逻辑推理，Codex 在代码生成速度上有优势，Cursor 对编辑器上下文的理解更深。当你创建 agent 的时候，你可以指定它跑在哪个 Runtime 上——就像一个项目经理知道团队里谁擅长什么。

一台机器上可以注册多个 Runtime，一个 workspace 可以有多个 Runtime（来自不同机器或不同工具）。Server 会维护一个 Runtime 注册表，知道当前哪些执行能力在线、哪些离线。当一个任务到来时，调度逻辑会根据 agent 指定的 Runtime 偏好来选择执行。

这个设计在你有多台机器时特别有用。比如你的台式机装了 GPU，跑得动大型模型推理；你的笔记本轻便但只装了你日常开发的工具。你可以把计算密集型 agent 指定到台式机的 Runtime，把代码审查 agent 留在笔记本。机器离线了，对应的 Runtime 就从注册表里消失，任务不会被分配到它——Server 不会傻等一台关机的电脑。

**目前 Multica 的 Runtime 主要是本地 daemon 模式。** 也就是说，执行能力绑定在你的物理机器上。未来可能会有远程 Runtime——比如托管在云服务器上的 agent 执行环境——但现阶段，"Runtime = 你的机器 × 你装好的工具"。

对于刚开始用的用户，不需要纠结于 Runtime 的细节。daemon 默认会用你安装的第一个可用工具创建一个 Runtime。等你开始用多种工具、或者多台机器协作时，Runtime 的概念自然会变得有意义。

一个实用建议：给不同类型的任务创建不同 Runtime 偏好的 agent。比如，创建一个"代码审查"agent 指定用 Claude Code（看重推理能力），创建一个"快速修复"agent 指定用 Codex（看重速度）。这不是过度工程——当你有 5 个以上的 agent 在不同任务上协作时，合理的 Runtime 分配可以显著提升整体吞吐。

关键是不要把所有的 agent 当成同一种能力。它们跑在不同的 Runtime 上，背后是不同的工具和不同的机器。理解这一点，你才能把 Multica 从一个"自动执行任务的工具"变成一个**可编排的 AI 执行集群**。
