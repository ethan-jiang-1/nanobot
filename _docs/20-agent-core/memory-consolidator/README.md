# memory-consolidator

本目录用于拆解 MemoryConsolidator 与 MemoryStore 的记忆收敛机制。

## 本主题要回答的问题

- 何时触发记忆归纳，触发阈值是什么
- 归纳边界如何选，为什么按 user turn 截断
- 归纳失败时如何降级，如何保证可恢复
- MEMORY.md 与 HISTORY.md 分别承担什么角色

## 建议文档拆分

- `01-memory-store-model-and-files.md`：MEMORY.md 与 HISTORY.md 的职责分工
- `02-save-memory-tool-protocol.md`：save_memory 工具调用协议与参数归一化
- `03-trigger-policy-and-token-estimation.md`：触发阈值、token 估算、预压缩时机
- `04-boundary-selection-and-rounds.md`：按 user turn 切分与多轮收敛机制
- `05-failure-fallback-and-raw-archive.md`：失败计数、降级写入、可恢复性

## 产出数量

- 预计 5 篇，覆盖 `memory.py` 的策略层与存储层

## 分析抓手

- `maybe_consolidate_by_tokens`
- `pick_consolidation_boundary`
- `MemoryStore.consolidate`
