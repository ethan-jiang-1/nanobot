# gateway 关停顺序与资源 drain

## 起点与终点

- 起点：gateway 运行协程进入 `finally`
- 终点：MCP、后台任务、运行时服务、channels 全部关闭

## 固定关停顺序

当前顺序是：

1. `await agent.close_mcp()`
2. `heartbeat.stop()`
3. `cron.stop()`
4. `agent.stop()`
5. `await channels.stop_all()`

源码锚点：

- gateway 主流程：[commands.py](file:///Users/bowhead/nanobot/nanobot/cli/commands.py)

## 为什么先 close_mcp

`close_mcp` 会先等待 agent 内部后台任务收尾，再关闭 MCP 资源栈，能减少“任务没写完资源先断开”的风险。

源码锚点：

- close_mcp 实现：[loop.py](file:///Users/bowhead/nanobot/nanobot/agent/loop.py)

## channel 侧停止语义

`ChannelManager.stop_all` 先停 outbound dispatcher，再逐个 channel `stop()`，避免停机期间继续发送消息。

源码锚点：

- channel stop 实现：[manager.py](file:///Users/bowhead/nanobot/nanobot/channels/manager.py)
