---
title: 从分配 issue 到看到结果——一次任务的完整旅程
date: 2026-06-23 11:00:00
categories:
  - 人工智能
tags:
  - Multica
  - 任务编排
  - Agent
---

前两篇讲了 Multica 是什么、三层如何分工。这一篇把镜头拉近，跟踪一个任务的完整生命周期——从你点击"分配"到评论里出现结果。

整个过程只有五个状态：**queued → dispatched → running → completed（或 failed）**。

你在 Multica Web 端把 `SWO-518` 分配给了名为"Blog沉淀"的 agent。浏览器发送一个 HTTP 请求到 Server。Server 做了三件事：更新 issue 的 assignee，在任务队列中插入一条记录，把状态设为 `queued`。同时通过 WebSocket 告诉前端："assignee 变了，任务已入队"。你在页面上看到 agent 名字出现，状态栏显示"排队中"。

与此同时，你本机的 daemon 在后台每几秒轮询一次 Server。它发现了这条新任务——assignee 是自己（这个 workspace 注册的 agent），状态是 `queued`。daemon 发送一个领取请求，Server 把状态改为 `dispatched`，确保不会有另一个 daemon 重复领取。

现在状态是 `dispatched`。daemon 开始准备执行环境：创建一个隔离的工作目录（类似 `git worktree`），把 issue 的标题和描述作为 prompt 的一部分注入，然后调用你配置好的 AI 编程工具——比如 Claude Code。状态变为 `running`。

Claude Code 在隔离目录里读取仓库代码，理解 issue 要求，开始改文件。它可能会运行测试、检查 lint、甚至自己执行 git 操作。每一步的中间输出可以通过 daemon 回传到 Server，但真正重要的是最终结果。

执行完成后，Claude Code 退出。daemon 收集输出（包括修改了哪些文件、命令行输出、错误信息如果有的话），打包发送给 Server。Server 把结果保存为一条 issue 评论，任务状态变为 `completed`。WebSocket 再次推送——你的前端实时看到评论里出现了一长段 agent 的输出，任务标记为绿色对勾。

如果中途出了问题——比如 Claude Code 调用了不存在的文件、API 超时、或者 agent 自己判定无法完成任务——daemon 会把错误信息回传，任务状态变为 `failed`。你看到的是红色标记，评论里有失败原因。

理解这个生命周期不是为了欣赏它的精巧，而是为了**建立排障心智**。如果你的任务一直卡在 `queued`：daemon 没在线。卡在 `dispatched` 但迟迟不进 `running`：可能是工作目录创建失败、或者工具不可用。`failed` 之后怎么办？读 agent 的评论，修正 issue 描述，重新触发。

还有一个你可能没注意到的细节：整个过程前端不需要刷新。当你把浏览器开着 Multica 页面时，WebSocket 会静默地推送每一次状态变化。这意味着你可以分配一个任务，然后离开去开会——回来的时候，结果已经在评论里了。

这种"fire and forget"的体验，底层靠的是任务队列的异步模型。Server 从来不阻塞等待 daemon 完成——它只管入队、记录状态变化、推送通知。daemon 和 AI 工具跑多久，Server 都不在意。这跟传统 Web 应用里"等一个 API 返回"的心智完全不同，更像是消息队列的消费者模型。

下一次你的 agent 任务卡住时，不用猜——顺着这五个状态排查就行。
