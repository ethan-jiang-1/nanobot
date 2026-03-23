# Agent run 循环与入站分发

## 起点与终点

- 起点：`agent.run()` 启动
- 终点：每条 inbound 要么进入命令处理，要么派发 `_dispatch`

## run 主循环

`run` 采用 `wait_for(..., timeout=1.0)` 轮询 inbound：

- 正常拿到消息：继续分流
- timeout：继续下一轮
- 消费异常：记录日志后继续

源码锚点：

- run 主循环：[loop.py:L257-L287](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L257-L287)

## 命令优先分流

在 `_dispatch` 之前先做短路：

- `/stop`、`/restart`、`/status` 走 priority 路由
- 其他 -> 创建 `_dispatch` task

priority 命令在 `run` 里直接 `dispatch_priority`，不会进入全局处理锁；其余命令在 `_process_message` 内再走 `dispatch`。

同时用 `_active_tasks[session_key]` 记录任务，便于 `/stop` 按会话精准取消。

源码锚点：

- 分流与任务登记：[loop.py:L328-L337](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L328-L337)
- priority 路由判定与分发：[loop.py:L328-L335](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L328-L335)
- router 优先/常规分层：[router.py:L27-L84](file:///Users/bowhead/nanobot/nanobot/command/router.py#L27-L84)

## 相关测试

- `/stop` 取消活动任务：[test_task_cancel.py:L42-L65](file:///Users/bowhead/nanobot/tests/test_task_cancel.py#L42-L65)
- `/restart` 在 run 层被拦截：[test_restart_command.py:L47-L67](file:///Users/bowhead/nanobot/tests/test_restart_command.py#L47-L67)
- 外部取消不被吞掉：[test_restart_command.py:L69-L79](file:///Users/bowhead/nanobot/tests/test_restart_command.py#L69-L79)
