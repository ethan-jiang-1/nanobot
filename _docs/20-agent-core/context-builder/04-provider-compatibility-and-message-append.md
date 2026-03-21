# provider 兼容与消息追加规则

## 目标

这篇说明 ContextBuilder 如何通过消息形状约束，规避不同 provider 的常见兼容问题。

## 关键兼容策略：单条 user 合并

`build_messages` 把 runtime context 与本轮用户输入强制合并成一条 `role=user` 消息。

原因很直接：某些 provider 对连续同 role 消息更严格，可能拒绝或降级处理。

源码锚点：

- 合并逻辑与注释：[context.py:L134-L140](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L134-L140)

相关测试：

- 最后一条必须是 merged user：[test_context_prompt_cache.py:L50-L73](file:///Users/bowhead/nanobot/tests/test_context_prompt_cache.py#L50-L73)

## assistant 消息追加

`add_assistant_message` 不手写字典，而是统一走 `build_assistant_message`，带来三个收益：

- 在有/无 tool_calls 时都产出一致结构
- 可附带 `reasoning_content` 与 `thinking_blocks`
- 便于 provider 适配逻辑统一维护

源码锚点：

- 追加入口：[context.py:L181-L195](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L181-L195)
- 构造器：[helpers.py:L81-L97](file:///Users/bowhead/nanobot/nanobot/utils/helpers.py#L81-L97)

## tool 消息追加

`add_tool_result` 生成标准 tool 消息字段：

- `role=tool`
- `tool_call_id`
- `name`
- `content`

这与 `_run_agent_loop` 里的 tool call id 一一配对，是后续 Session 合法性修复的基础。

源码锚点：

- [context.py:L173-L179](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L173-L179)
- 触发方：[loop.py:L223-L233](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L223-L233)

## 与 SessionManager 的耦合边界

ContextBuilder 负责“当前轮的消息形状正确”，SessionManager 负责“历史窗口切片后仍然合法”。

即使前者正确，窗口裁剪也可能制造 orphan tool result；因此后者仍需 `_find_legal_start` 兜底。

相关实现：

- 历史合法化：[manager.py:L40-L87](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L40-L87)
- 回归测试：[test_session_manager_history.py:L36-L146](file:///Users/bowhead/nanobot/tests/test_session_manager_history.py#L36-L146)

## 设计结论

- ContextBuilder 的兼容目标是“构建时不违规”
- SessionManager 的兼容目标是“切片后不违规”
- 两层配合，才能在多 provider 下保持稳定对话
