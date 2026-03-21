# 工具上下文注入与回合语义

## 起点与终点

- 起点：`_process_message` 进入一次新回合
- 终点：工具拿到当前会话上下文，回合结束后决定是否发送默认最终回复

## 上下文注入机制

`AgentLoop` 会在每次处理消息时调用 `_set_tool_context(channel, chat_id, message_id)`，把路由上下文写入需要的工具：

- `message`：用于默认发回当前会话，可携带 `message_id`
- `spawn`：用于子代理回传时定位来源会话
- `cron`：用于计划任务投递到当前会话

源码锚点：

- 上下文注入方法：[loop.py:L159-L165](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L159-L165)
- 注入调用点（普通消息）：[loop.py:L426-L429](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L426-L429)
- 注入调用点（system 消息）：[loop.py:L378-L379](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L378-L379)

## message 工具的回合抑制语义

`message` 工具在当前回合若向默认目标发送成功，会标记 `_sent_in_turn=True`。  
回合收尾时，`AgentLoop` 检查该标志并返回 `None`，抑制“再发一条默认最终回复”。

这避免“工具已主动发消息 + 主循环再重复发一次”的重复输出。

源码锚点：

- 回合开始重置标志：[loop.py:L427-L430](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L427-L430)
- 回合结束抑制判断：[loop.py:L458-L460](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L458-L460)
- message 工具标志设置：[message.py:L102-L107](file:///Users/bowhead/nanobot/nanobot/agent/tools/message.py#L102-L107)

## 设计边界

- 该机制只影响默认最终回复，不影响进度消息
- 抑制条件是“发送到默认上下文成功”，跨会话发送不触发抑制
