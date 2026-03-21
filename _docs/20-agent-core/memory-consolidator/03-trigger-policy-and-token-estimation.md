# 触发策略与 token 估算

## 目标

这篇说明 `maybe_consolidate_by_tokens` 在什么条件下触发归纳，以及 token 估算是如何做的。

## 触发前置条件

以下情况直接不触发：

- session 没有消息
- `context_window_tokens <= 0`
- 估算 token <= 0
- 估算值 `< context_window_tokens`

源码锚点：

- [memory.py:L302-L322](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L302-L322)

## 核心阈值模型

触发阈值不是单一值，而是两段：

- 触发线：`estimated >= context_window_tokens`
- 收敛目标：`target = context_window_tokens // 2`

一旦触发，会继续归纳直到小于半窗口，而不是只降到“刚好不超”。

源码锚点：

- [memory.py:L309-L325](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L309-L325)

相关测试：

- 低于触发线不归纳：[test_loop_consolidation_tokens.py:L28-L36](file:///Users/bowhead/nanobot/tests/test_loop_consolidation_tokens.py#L28-L36)
- 触发后继续到半阈值：[test_loop_consolidation_tokens.py:L118-L152](file:///Users/bowhead/nanobot/tests/test_loop_consolidation_tokens.py#L118-L152)

## token 估算方法

`estimate_session_prompt_tokens` 不是估算消息数组本身，而是构造一份真实“即将发给模型”的 probe prompt：

1. 取 `session.get_history(max_messages=0)`
2. 补一条 `[token-probe]` 当前消息
3. 调用 provider 侧 token 估算链路

源码锚点：

- [memory.py:L276-L291](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L276-L291)

## 与 AgentLoop 的时机关系

在 `_process_message` 中，归纳检查发生在真正调用 LLM 前。  
这避免把超长上下文直接喂给 provider 才报错。

源码锚点：

- 预检查调用点：[loop.py:L424-L424](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L424-L424)

相关测试：

- 归纳先于 LLM 调用：[test_loop_consolidation_tokens.py:L155-L190](file:///Users/bowhead/nanobot/tests/test_loop_consolidation_tokens.py#L155-L190)

## 上限控制

单次 `maybe_consolidate_by_tokens` 最多跑 `_MAX_CONSOLIDATION_ROUNDS = 5` 轮，防止极端场景无限归纳。

源码锚点：

- [memory.py:L225-L225](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L225-L225)
