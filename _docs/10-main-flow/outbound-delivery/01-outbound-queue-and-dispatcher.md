# outbound 队列与 dispatcher

## 起点与终点

- 起点：agent 或工具发布 `OutboundMessage`
- 终点：dispatcher 将消息路由到目标 channel 的 `send`

## 队列层语义

`MessageBus` 的 outbound 只有两件事：

- `publish_outbound`：入队
- `consume_outbound`：出队

业务分发逻辑不在 bus 层。

源码锚点：

- outbound API：[queue.py:L27-L33](file:///Users/bowhead/nanobot/nanobot/bus/queue.py#L27-L33)

## dispatcher 主循环

`ChannelManager._dispatch_outbound` 持续消费队列并分发：

- 先按 `_progress/_tool_hint` 与全局配置做过滤
- 再按 `msg.channel` 查目标 channel
- 命中则 `await channel.send(msg)`，否则 warning

源码锚点：

- 分发循环：[manager.py:L113-L142](file:///Users/bowhead/nanobot/nanobot/channels/manager.py#L113-L142)

## 发送来源

- Agent 最终回复：`_dispatch`/`_process_message` 路径 publish
- 进度消息：`_bus_progress` publish
- message 工具：直接复用同一个 outbound callback

源码锚点：

- agent 发布：[loop.py:L323-L338](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L323-L338)
- progress 发布：[loop.py:L439-L445](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L439-L445)
- message tool 发送：[message.py:L92-L107](file:///Users/bowhead/nanobot/nanobot/agent/tools/message.py#L92-L107)

