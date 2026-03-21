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

- `/stop` -> `_handle_stop`
- `/restart` -> `_handle_restart`
- 其他 -> 创建 `_dispatch` task

同时用 `_active_tasks[session_key]` 记录任务，便于 `/stop` 按会话精准取消。

源码锚点：

- 分流与任务登记：[loop.py:L278-L287](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L278-L287)
- stop 处理：[loop.py:L288-L302](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L288-L302)
- restart 处理：[loop.py:L304-L316](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L304-L316)

## 相关测试

- `/stop` 取消活动任务：[test_task_cancel.py:L42-L65](file:///Users/bowhead/nanobot/tests/test_task_cancel.py#L42-L65)
- `/restart` 在 run 层被拦截：[test_restart_command.py:L47-L67](file:///Users/bowhead/nanobot/tests/test_restart_command.py#L47-L67)
- 外部取消不被吞掉：[test_restart_command.py:L69-L79](file:///Users/bowhead/nanobot/tests/test_restart_command.py#L69-L79)

