# AgentLoop 工具执行迭代闭环

## 起点与终点

- 起点：`_run_agent_loop(initial_messages)` 被调用
- 终点：返回 `final_content / tools_used / messages`

## 单轮执行序列

每轮核心动作：

1. 从 registry 获取 `tool_defs`
2. 调用 `provider.chat_with_retry(messages, tools=tool_defs)`
3. 若有 `tool_calls`：先写 assistant 消息，再逐个执行工具并写 tool 结果
4. 若无 `tool_calls`：写 assistant 文本并结束

源码锚点：

- 主循环与工具执行：[loop.py:L184-L255](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L184-L255)

## 进度与可观测性

在模型返回 tool_calls 时，loop 会发两类进度：

- 清洗 `<think>` 后的文本进度
- 简化版 tool hint（如 `web_search("...")`）

源码锚点：

- 进度回调：[loop.py:L206-L214](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L206-L214)

## 终止条件

- 模型给出无 tool_call 的最终文本
- 或达到 `max_iterations`
- 或 provider 返回 `finish_reason="error"`，走错误文案收敛

源码锚点：

- error 收敛与上限收敛：[loop.py:L233-L253](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L233-L253)
