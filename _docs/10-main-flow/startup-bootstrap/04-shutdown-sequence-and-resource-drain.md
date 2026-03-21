# 关闭顺序与资源回收

## 起点与终点

- 起点：gateway `run()` 进入 `finally`
- 终点：MCP、heartbeat、cron、agent、channels 全部停止

## 关闭顺序

当前顺序是：

1. `await agent.close_mcp()`
2. `heartbeat.stop()`
3. `cron.stop()`
4. `agent.stop()`
5. `await channels.stop_all()`

源码锚点：

- gateway finally：[commands.py:L671-L677](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L671-L677)

## 为什么先 `close_mcp`

`close_mcp` 不只是关 MCP，它还会先 drain `AgentLoop` 的后台任务：

- `await gather(*_background_tasks, return_exceptions=True)`
- 清空 `_background_tasks`
- 关闭 `_mcp_stack`

这保证了“后台归档尽量收尾完成，再释放连接”。

源码锚点：

- `close_mcp`：[loop.py:L340-L350](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L340-L350)

## Channel 侧收尾

`ChannelManager.stop_all` 会先取消 outbound dispatcher，再逐个 `await channel.stop()`，避免继续出站发送。

源码锚点：

- `stop_all`：[manager.py:L93-L112](file:///Users/bowhead/nanobot/nanobot/channels/manager.py#L93-L112)

## 风险控制点

- `close_mcp` 使用 `return_exceptions=True`，单个后台任务失败不阻塞整体关停
- dispatcher 取消后不会继续消费 outbound，减少重复/迟到发送

