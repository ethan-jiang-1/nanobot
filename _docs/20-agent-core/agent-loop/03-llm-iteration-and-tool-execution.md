# LLM 迭代与工具执行闭环

## 目标

这篇解释 AgentLoop 的核心智能闭环：`LLM -> tool_calls -> tool_result -> LLM`。

## 入口与输出

核心函数是 `_run_agent_loop(initial_messages)`，输出三项：

- `final_content`：最终回复文本
- `tools_used`：本轮用过的工具名列表
- `messages`：完整消息链（含 assistant/tool）

源码锚点：

- `_run_agent_loop`：[loop.py:L184-L255](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L184-L255)

## 单轮逻辑

每次迭代都做同样三步：

1. 拉取当前工具定义 `tool_defs`
2. 调用 `provider.chat_with_retry(messages, tools, model)`
3. 根据 `response.has_tool_calls` 分支

这意味着工具集合是“运行时读取”，而非静态写死。

## 有工具调用时

当模型返回 tool_calls：

- 先把 assistant 消息（含 tool_calls 元数据）写入消息链
- 逐个执行工具 `self.tools.execute(name, arguments)`
- 将工具输出以 `role=tool` 追加回消息链

然后进入下一轮，让模型读取工具结果继续推理。

源码锚点：

- tool call 分支：[loop.py:L206-L233](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L206-L233)

## 无工具调用时

当模型直接给出文本答案：

- 先清理 `<think>...</think>` 隐式思维块
- 若 `finish_reason == "error"`，返回错误内容并中断
- 否则写入 assistant 消息并结束迭代

这避免把错误响应持续灌入历史，降低后续 400 循环风险。

源码锚点：

- 非工具分支：[loop.py:L233-L246](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L233-L246)

## 迭代上限

`max_iterations` 是硬上限，超过后给出“任务过大，建议拆分”的提示。

作用：

- 防止工具链无限循环
- 给用户明确失败语义而不是沉默

源码锚点：

- 上限分支：[loop.py:L248-L253](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L248-L253)

## 进度回调

函数支持 `on_progress`，用于把中间推理提示/工具提示实时回传到前端 channel。

这也是 UI 能显示“正在调用某工具”的基础。

源码锚点：

- progress 处理：[loop.py:L207-L214](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L207-L214)
