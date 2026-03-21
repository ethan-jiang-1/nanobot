# agent-loop

本目录用于拆解 AgentLoop 这一核心调度引擎。

## 本主题要回答的问题

- 一条消息进入后，完整执行路径是什么
- 何时进入工具调用循环，何时结束
- `/stop`、`/restart`、`/new` 如何影响执行态
- 并发任务与全局锁如何协同

## 建议文档拆分

- `01-lifecycle-and-entry.md`：初始化、启动、关闭
- `02-iteration-and-tool-calls.md`：LLM-Tool-LLM 迭代机制
- `03-cancellation-and-recovery.md`：停止、重启、异常回退

## 分析抓手

- `run` / `_dispatch` / `_process_message`
- `_run_agent_loop` 的迭代退出条件
- `_save_turn` 的会话写入与截断策略
