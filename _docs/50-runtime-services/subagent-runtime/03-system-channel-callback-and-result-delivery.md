# system 回注与结果投递链路

## 起点与终点

- 起点：子代理得到 `final_result` 或错误信息
- 终点：主代理把结果重写为用户可读回复并发回原会话

## 回注消息构造

`_announce_result` 不直接发 outbound，而是构造 system inbound：

- `channel="system"`
- `sender_id="subagent"`
- `chat_id` 编码为 `origin_channel:origin_chat_id`
- content 包含任务、结果与“自然化总结”指令

源码锚点：

- 回注消息构造：[subagent.py:L168-L197](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L168-L197)

## AgentLoop 的 system 分支

`_process_message` 对 `channel="system"` 走专门分支：

- 从 `chat_id` 反解原目标 channel/chat_id
- `sender_id == "subagent"` 时把当前消息角色设为 assistant
- 调用 `_run_agent_loop` 生成对用户友好的最终文本
- 结果以 `OutboundMessage(channel, chat_id, content)` 返回

源码锚点：

- system 分支：[loop.py:L370-L393](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L370-L393)

## 为什么走 system 入站而不是直接 outbound

这条设计把“子代理原始输出”与“用户最终看到的文本”解耦：

- 主代理可统一口径、压缩细节
- 仍复用会话历史、上下文与工具能力
- 子代理实现可变更，不影响用户交互面

## 可见结果语义

system 分支最终返回：

- 子代理有文本时：返回主代理整理后的内容
- 子代理无文本时：回退 `"Background task completed."`

源码锚点：

- 回退文案：[loop.py:L391-L393](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L391-L393)
