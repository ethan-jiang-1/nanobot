# 命令路径与控制流分支

## 目标

这篇单独拆解 AgentLoop 里几条高优先级命令分支，说明它们如何影响会话与任务状态。

## `/new`：重置当前会话

行为：

- 取出当前会话尚未归纳的快照
- `session.clear()` 清空消息与 consolidation offset
- 保存并失效缓存
- 将旧快照异步归档到 memory

结果是：会话上下文立即“重开”，但历史不直接丢失。

源码锚点：

- `/new` 分支：[loop.py:L401-L413](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L401-L413)

## `/help`：命令说明输出

行为比较直接：返回固定命令列表，不触发模型调用。

源码锚点：

- `/help` 分支：[loop.py:L413-L423](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L413-L423)

## `/stop`：取消会话活动任务

在 `run` 主循环中命中 `/stop` 后，会进入 `_handle_stop`：

- 取消 `_active_tasks[session_key]` 内所有未完成任务
- 同步取消该会话下的子代理任务
- 汇总取消数量并向用户回告

源码锚点：

- 命中点：[loop.py:L278-L281](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L278-L281)
- 实现：[loop.py:L288-L302](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L288-L302)

## `/restart`：进程内重启

流程：

1. 先发一条 `Restarting...`
2. 延时 1 秒
3. 通过 `os.execv` 以 `python -m nanobot` 方式替换当前进程

这是“原地重启”，不是启动并行新进程。

源码锚点：

- 命中点：[loop.py:L281-L283](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L281-L283)
- 实现：[loop.py:L304-L316](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L304-L316)

## system channel 的特殊路径

`msg.channel == "system"` 时走专门分支，典型用于子代理回传：

- chat_id 可携带原会话信息
- 可按 sender 决定当前消息 role 是 assistant 还是 user
- 结束后默认回包 `Background task completed.`

源码锚点：

- system 路径：[loop.py:L370-L393](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L370-L393)

## 控制流设计原则

- 控制命令尽量短路，不占用模型额度
- 会话状态变更尽量可恢复（归档优先于删除）
- 对用户始终给出可见反馈，而不是静默操作
