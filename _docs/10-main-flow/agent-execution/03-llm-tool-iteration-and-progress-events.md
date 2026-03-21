# LLM-工具迭代与进度事件

## 起点与终点

- 起点：`_process_message` 调用 `_run_agent_loop(initial_messages)`
- 终点：得到 `final_content / tools_used / all_messages`

## 迭代闭环

`_run_agent_loop` 每轮都执行：

1. 读取当前工具定义
2. 调用 `provider.chat_with_retry`
3. 若有 tool_calls：写 assistant/tool 并进入下一轮
4. 若无 tool_calls：写 assistant 并结束

源码锚点：

- 主循环：[loop.py:L184-L255](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L184-L255)

## 进度消息机制

有工具调用时会通过回调推送两类进度：

- 普通进度：清洗过 `<think>` 的文本
- tool hint：格式化后的调用提示（如 `read_file("...")`）

在默认路径里，这些回调会被包装成 outbound 消息并打上元数据：

- `_progress = True`
- `_tool_hint = True/False`

源码锚点：

- progress 生成：[loop.py:L207-L214](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L207-L214)
- bus progress 包装：[loop.py:L439-L445](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L439-L445)

## 相关测试

- progress 隐去 `<think>` 与仅保留 tool hint：[test_message_tool_suppress.py:L89-L116](file:///Users/bowhead/nanobot/tests/test_message_tool_suppress.py#L89-L116)

