# `_process_message` 主执行时序

## 起点与终点

- 起点：`_dispatch` 在全局锁内调用 `_process_message(msg)`
- 终点：返回 `OutboundMessage` 或 `None`（特定路径）

## 常规路径时序

1. 定位会话 `session = sessions.get_or_create(key)`
2. 命令短路（`/new`、`/help`）
3. 预先执行 token 归纳检查
4. 设置工具上下文（channel/chat_id/message_id）
5. 构建 `initial_messages`
6. 执行 `_run_agent_loop`
7. `_save_turn` 回写并 `sessions.save`
8. 异步调度 post-turn 归纳
9. 根据 message tool 是否同目标发送，决定返回正文还是 `None`

源码锚点：

- 常规路径：[loop.py:L394-L466](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L394-L466)

## system 路径

`msg.channel == "system"` 时会把 `chat_id` 解析回真实来源，走专门分支并默认回包：

- 子代理回传时 `current_role` 用 assistant
- 其他 system 消息用 user

源码锚点：

- system 分支：[loop.py:L370-L393](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L370-L393)

## 相关测试

- `_dispatch` 正常回包：[test_task_cancel.py:L101-L112](file:///Users/bowhead/nanobot/tests/test_task_cancel.py#L101-L112)
- message tool 抑制最终回包：[test_message_tool_suppress.py:L26-L49](file:///Users/bowhead/nanobot/tests/test_message_tool_suppress.py#L26-L49)

