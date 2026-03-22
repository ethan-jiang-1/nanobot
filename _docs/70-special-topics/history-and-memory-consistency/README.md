# history-and-memory-consistency

本目录聚焦长会话运行下的一致性问题：历史写入卫生、切片合法性、记忆归纳与失败恢复。

## 本主题要回答的问题

- runtime 上下文为什么不能直接持久化
- 历史裁切时如何避免 tool 调用孤儿片段
- token 压力下如何分轮归纳而不破坏语义
- 归纳失败后如何保证信息可恢复、可追溯

## 建议文档拆分

- `01-runtime-context-stripping-and-history-hygiene.md`：回合写回卫生策略
- `02-tool-call-boundary-and-history-slicing.md`：历史切片合法性
- `03-token-pressure-consolidation-policy.md`：归纳触发与边界选择
- `04-failure-fallback-and-raw-archive-recovery.md`：失败降级与恢复路径

## 分析抓手

- `AgentLoop._save_turn`
- `Session.get_history`
- `MemoryConsolidator.maybe_consolidate_by_tokens`
- `MemoryStore.consolidate`
