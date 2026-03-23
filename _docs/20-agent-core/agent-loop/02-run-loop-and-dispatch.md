# run 循环与分发机制

## 目标

这篇聚焦 AgentLoop 在运行态如何消费消息、分派任务，以及如何在并发下保持可控。

## 事件主循环

`run` 的核心是一个 `while self._running` 循环，内部通过 `wait_for(..., timeout=1.0)` 轮询消息总线：

- 有消息：进入命令判断与任务分发
- 超时：继续下一轮
- 非预期异常：记录 warning 并继续

这样可以避免长期阻塞在 `consume_inbound`，让停止信号更快生效。

源码锚点：

- 主循环：[loop.py:L263-L287](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L263-L287)

## 命令分流

在进入业务处理前，先做指令短路：

- `/stop`、`/restart`、`/status` -> priority dispatch
- 其他 -> 创建异步任务 `_dispatch`

这条分支是“控制面优先”的体现：先处理系统控制，再处理普通对话。

源码锚点：

- 分支入口：[loop.py:L328-L335](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L328-L335)
- priority 分发：[loop.py:L328-L335](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L328-L335)
- router 分层策略：[router.py:L27-L84](file:///Users/bowhead/nanobot/nanobot/command/router.py#L27-L84)

## active_tasks 的作用

每个会话 key 维护一个任务列表 `_active_tasks[session_key]`，用于：

- `/stop` 时精准取消当前会话相关任务
- 避免全局 stop 误杀其他会话任务

任务完成后会通过回调自动从列表里移除，避免内存堆积。

源码锚点：

- 任务登记与回收：[loop.py:L335-L337](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L335-L337)

## `_dispatch` 与全局处理锁

`_dispatch` 内部使用 `self._processing_lock` 包裹 `_process_message`，意味着同一时刻只允许一个消息进入核心处理流程。

好处：

- 降低会话状态竞争概率
- 让 session 写入与记忆收敛更可预测

代价：

- 并发吞吐不是最大化，而是优先一致性与稳定性

源码锚点：

- `_dispatch`：[loop.py:L339-L375](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L339-L375)

## 分发后的回传语义

`_dispatch` 中有三种回传路径：

- 有响应对象：直接 publish outbound
- 返回 `None` 且 channel=cli：发空串触发 CLI 侧流控结束
- 异常：回传统一错误文案

这解释了为什么某些工具路径执行后你会看到“无最终正文但流程正常结束”。

若消息声明 `_wants_stream`，`_dispatch` 还会走增量回包协议：

- `_stream_delta`：内容增量分片
- `_stream_end`：一段 streaming 结束，`_resuming` 表示是否继续工具轮
- `_streamed`：最终正文已通过增量发送，渠道侧应跳过重复发送

## 与消息总线的边界

- AgentLoop 只负责消费/发布，不关心 bus 内部实现
- 这使得 channel 层和 agent 层解耦，可独立演进
