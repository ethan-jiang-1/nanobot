# agent-execution

本目录聚焦 Agent 核心执行段：消息进入 `_process_message` 后如何构建上下文、调用模型、执行工具并回写状态。

## 本主题要回答的问题

- `_process_message` 的完整时序是什么
- context 是如何注入 runtime 元数据并与历史拼接
- LLM 与工具调用如何形成多轮闭环
- 回合结束后 session 与后置归纳如何处理

## 建议文档拆分

- `01-process-message-high-level-sequence.md`：`_process_message` 高层流程
- `02-context-build-and-runtime-metadata.md`：`ContextBuilder` 组包策略
- `03-llm-tool-iteration-and-progress-events.md`：`_run_agent_loop` 与进度事件
- `04-session-writeback-and-post-turn-consolidation.md`：`_save_turn` 与归纳调度

## 产出数量

- 预计 4 篇，覆盖 Agent 主执行链路

