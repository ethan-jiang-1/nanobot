# consolidation offset 与 history 视图协作

## 目标

这篇专门讲 `last_consolidated` 如何与 `get_history`、MemoryConsolidator 协同，形成“归纳但不改写消息”的机制。

## `last_consolidated` 的定位

它表示“已被归纳进 memory 文件的消息数量”，而不是“要删除的消息数量”。

所以：

- `messages` 仍保存完整 append-only 历史
- `get_history` 只从 offset 之后取给 LLM

源码锚点：

- 字段定义：[manager.py:L33-L33](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L33-L33)
- history 起点：[manager.py:L71-L72](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L71-L72)

## offset 何时推进

真正推进 offset 的地方在 MemoryConsolidator：

1. 选出 chunk 并归纳成功
2. `session.last_consolidated = end_idx`
3. `sessions.save(session)` 持久化

源码锚点：

- [memory.py:L336-L353](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L336-L353)

## 为什么这比“删除旧消息”更稳

- 不破坏会话原始轨迹
- 可以重复验证归纳策略正确性
- 出现归纳回退时仍有原始历史

同时通过 offset 限制模型看到的窗口，避免 prompt 膨胀。

## 持久化保证

`save` 会把 `last_consolidated` 写入 metadata；`_load` 会在重载时恢复该值。

源码锚点：

- 写入：[manager.py:L197-L204](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L197-L204)
- 读取：[manager.py:L174-L178](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L174-L178)

## 测试覆盖

- 新会话 offset 初始为 0  
  [test_consolidate_offset.py:L62-L66](file:///Users/bowhead/nanobot/tests/test_consolidate_offset.py#L62-L66)
- save/load 后 offset 保持  
  [test_consolidate_offset.py:L67-L77](file:///Users/bowhead/nanobot/tests/test_consolidate_offset.py#L67-L77)
- clear 会重置 offset  
  [test_consolidate_offset.py:L78-L86](file:///Users/bowhead/nanobot/tests/test_consolidate_offset.py#L78-L86)
- `get_history` 不会修改原 messages 列表  
  [test_consolidate_offset.py:L131-L142](file:///Users/bowhead/nanobot/tests/test_consolidate_offset.py#L131-L142)
