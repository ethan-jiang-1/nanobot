# 边界选择与多轮收敛

## 目标

这篇解释 MemoryConsolidator 为什么按 user turn 选择边界，以及多轮归纳如何推进 `last_consolidated`。

## `pick_consolidation_boundary` 的规则

边界选择核心点：

- 从 `session.last_consolidated` 开始扫描
- 按消息 token 逐步累计 `removed_tokens`
- 只在遇到“下一条 user 消息”时允许切边界
- 优先返回“满足 tokens_to_remove 的最早边界”
- 若全程不足，也返回最后一个可用 user 边界

源码锚点：

- [memory.py:L254-L274](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L254-L274)

## 为什么按 user 边界切

如果在 assistant/tool 中间切，容易破坏对话语义完整性，甚至制造工具调用上下文断裂。  
按 user turn 切能更稳定地保留“完整回合”。

## 多轮归纳流程

每轮做四步：

1. 计算边界 `end_idx`
2. 截取 `chunk = messages[last_consolidated:end_idx]`
3. 调 `consolidate_messages(chunk)`
4. 成功后更新 `session.last_consolidated = end_idx` 并持久化

源码锚点：

- [memory.py:L323-L353](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L323-L353)

## `last_consolidated` 的意义

它不是删除消息，而是“逻辑游标”：

- 会话全量消息仍 append-only 保存
- `get_history` 时自动跳过已归纳前缀
- 归纳结果写入 memory 文件，不回写原消息列表

关联实现：

- session 字段与 history 切片：[manager.py:L33-L73](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L33-L73)

## 关键测试覆盖

- 归档到下一个 user 边界：[test_loop_consolidation_tokens.py:L57-L79](file:///Users/bowhead/nanobot/tests/test_loop_consolidation_tokens.py#L57-L79)
- 多轮直到目标满足：[test_loop_consolidation_tokens.py:L82-L115](file:///Users/bowhead/nanobot/tests/test_loop_consolidation_tokens.py#L82-L115)
- offset 持久化与重载：[test_consolidate_offset.py:L67-L77](file:///Users/bowhead/nanobot/tests/test_consolidate_offset.py#L67-L77)

## 无安全边界时

当找不到可用边界（例如消息太短或 tokens_to_remove 无法满足），`maybe_consolidate_by_tokens` 会提前返回，不做冒险切分。
