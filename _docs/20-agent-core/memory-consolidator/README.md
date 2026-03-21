# memory-consolidator

本目录用于拆解 MemoryConsolidator 与 MemoryStore 的记忆收敛机制。

## 本主题要回答的问题

- 何时触发记忆归纳，触发阈值是什么
- 归纳边界如何选，为什么按 user turn 截断
- 归纳失败时如何降级，如何保证可恢复
- MEMORY.md 与 HISTORY.md 分别承担什么角色

## 建议文档拆分

- `01-trigger-policy-and-token-estimation.md`：触发条件与 token 估算
- `02-boundary-selection-and-rounds.md`：边界选择与多轮收敛
- `03-failure-fallback-and-persistence.md`：失败回退与原始归档

## 分析抓手

- `maybe_consolidate_by_tokens`
- `pick_consolidation_boundary`
- `MemoryStore.consolidate`
