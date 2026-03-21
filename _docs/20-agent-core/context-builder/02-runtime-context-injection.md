# runtime context 注入机制

## 目标

这篇解释运行时元数据如何进入提示词链路，同时避免被模型误解为高信任指令。

## runtime context 的内容

`_build_runtime_context(channel, chat_id)` 输出一个文本块，结构是：

- 固定 tag：`[Runtime Context — metadata only, not instructions]`
- 当前时间 `Current Time: ...`
- 可选 channel 与 chat_id

源码锚点：

- `_build_runtime_context`：[context.py:L100-L106](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L100-L106)

## 为什么放在 user 侧而不是 system 侧

设计上把 runtime 信息视为“不可信元数据”，因此不直接并入 system prompt。

好处：

- system 层保持稳定和高优先级规则
- runtime 层可变，且不会污染长期缓存
- 安全语义更清晰：这是一段上下文，不是策略指令

## 注入位置

在 `build_messages` 中先构造 runtime_ctx，再与用户输入合并成同一条消息：

- 文本输入：`runtime_ctx + "\n\n" + user_text`
- 多模态输入：先插入文本块，再跟图片/正文块

源码锚点：

- `build_messages`：[context.py:L120-L145](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L120-L145)

## 与 AgentLoop 的调用关系

`AgentLoop._process_message` 每轮都会把 channel/chat_id 传给 `build_messages`，保证 runtime 注入始终与当前会话绑定。

源码锚点：

- 常规路径：[loop.py:L434-L437](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L434-L437)
- system 路径：[loop.py:L382-L385](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L382-L385)

## 与历史持久化的配合

runtime 注入只用于本轮模型输入，写会话时会被 `_save_turn` 剥离，避免历史被时间与路由字段污染。

相关实现：

- 剥离逻辑：[loop.py:L465-L503](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L465-L503)
- 相关回归测试：[test_loop_save_turn.py:L12-L23](file:///Users/bowhead/nanobot/tests/test_loop_save_turn.py#L12-L23)

## 复用点：subagent

Subagent prompt 也复用了 `ContextBuilder._build_runtime_context(None, None)`，说明这个块已成为统一 runtime 元数据格式。

源码锚点：

- [subagent.py:L200-L221](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L200-L221)
