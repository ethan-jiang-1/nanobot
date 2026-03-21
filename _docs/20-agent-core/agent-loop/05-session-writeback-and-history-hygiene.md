# 会话回写与历史卫生策略

## 目标

这篇解释 `_save_turn` 为什么是稳定性的关键点：它控制“写入什么历史”，而不是简单全量 append。

## `_save_turn` 的职责

输入是“本次处理后得到的 messages”，输出是“适合长期存储的 session.messages”。

它至少做四类清洗：

- 跳过前置 `skip` 区间，避免重复写历史
- 过滤仅包含 runtime context 的伪 user 消息
- 清洗/截断 tool 结果，避免巨大 payload 污染会话
- 兼容多模态，把图片变成稳定占位文本

源码锚点：

- `_save_turn`：[loop.py:L488-L578](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L488-L578)

## runtime context 剥离

`ContextBuilder` 会在 user 文本前注入 runtime 信息（时间、channel 等），这类内容用于本轮推理，不该长期沉淀。

`_save_turn` 里会按 `_RUNTIME_CONTEXT_TAG` 定位并剥离，若剥离后 user 消息为空则整条跳过。

相关测试：

- runtime-only 跳过：[test_loop_save_turn.py:L12-L23](file:///Users/bowhead/nanobot/tests/test_loop_save_turn.py#L12-L23)

## 多模态占位策略

对于 `image_url` 内容，不直接保存 base64，而是落盘为可读占位：

- 有 path 元信息：`[image: /path/to/file]`
- 无 path：`[image]`

这在保留“发生过图片输入”语义的同时，避免历史膨胀。

相关测试：

- 带 path 占位：[test_loop_save_turn.py:L25-L42](file:///Users/bowhead/nanobot/tests/test_loop_save_turn.py#L25-L42)
- 无 meta 占位：[test_loop_save_turn.py:L44-L61](file:///Users/bowhead/nanobot/tests/test_loop_save_turn.py#L44-L61)

## 工具结果截断

tool 消息可能非常长，`_save_turn` 使用 `_TOOL_RESULT_MAX_CHARS` 上限做截断，优先保证会话可持续增长。

相关测试确认了“16k 内不截断”的正向路径：

- [test_loop_save_turn.py:L63-L74](file:///Users/bowhead/nanobot/tests/test_loop_save_turn.py#L63-L74)

## 为什么这一步在 loop 层

不是在 SessionManager 统一处理，而是在 loop 写入前处理，原因是：

- loop 更清楚哪些字段是“运行时临时上下文”
- loop 有工具执行语义，知道哪些输出可能异常大
- SessionManager 保持通用存储层职责，避免耦合业务语义
