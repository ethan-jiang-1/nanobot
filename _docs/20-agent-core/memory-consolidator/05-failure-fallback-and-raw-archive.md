# 失败回退与原始归档

## 目标

这篇聚焦记忆归纳失败时的降级路径：如何既保证主流程不阻塞，又保证信息不丢。

## 失败计数机制

`MemoryStore` 维护 `_consecutive_failures`：

- 失败次数 `< 3`：返回 `False`，提示调用方本轮未归纳
- 达到阈值：触发 raw archive，返回 `True`
- raw archive 后失败计数清零

源码锚点：

- 阈值常量：[memory.py:L78-L78](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L78-L78)
- 失败分支：[memory.py:L201-L208](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L201-L208)

## raw archive 写入内容

`_raw_archive` 会把原始消息块直接写入 `HISTORY.md`，头部标记：

- 时间戳
- `[RAW]`
- 消息条数

后面跟 `_format_messages(messages)` 的原文展开。

源码锚点：

- [memory.py:L210-L219](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L210-L219)

## 回退策略设计意图

- LLM 归纳失败不应卡死用户主会话
- 失败不应让历史完全丢失
- 累计失败后自动进入“保底归档”确保可恢复

## 关键测试覆盖

- 连续失败三次触发 RAW：[test_memory_consolidation_types.py:L436-L454](file:///Users/bowhead/nanobot/tests/test_memory_consolidation_types.py#L436-L454)
- 成功后失败计数重置：[test_memory_consolidation_types.py:L456-L478](file:///Users/bowhead/nanobot/tests/test_memory_consolidation_types.py#L456-L478)

## `archive_messages` 的语义

`MemoryConsolidator.archive_messages` 通过重试调用 `consolidate_messages`，目标是“保证函数完成并返回”，而不是向上抛异常。  
它与 `_fail_or_raw_archive` 的组合形成了端到端容错闭环。

源码锚点：

- [memory.py:L293-L300](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L293-L300)

## 工程结论

- MEMORY 归纳属于“可降级能力”，不是“强一致事务”
- 通过 RAW fallback，把不可控的模型失败转化为可控的存储退化
