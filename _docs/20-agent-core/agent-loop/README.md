# agent-loop

本目录用于拆解 AgentLoop 这一核心调度引擎。

## 本主题要回答的问题

- 一条消息进入后，完整执行路径是什么
- 何时进入工具调用循环，何时结束
- `/stop`、`/restart`、`/new` 如何影响执行态
- 并发任务与全局锁如何协同

## 建议文档拆分

- `01-lifecycle-and-wiring.md`：对象装配、默认工具注册、MCP 连接
- `02-run-loop-and-dispatch.md`：事件循环、消息消费、全局处理锁
- `03-llm-iteration-and-tool-execution.md`：模型迭代、工具调用、退出条件
- `04-command-paths-and-control-flow.md`：`/new` `/help` `/stop` `/restart` 分支
- `05-session-writeback-and-history-hygiene.md`：`_save_turn`、截断、运行时上下文剥离
- `06-background-tasks-and-shutdown.md`：后台归档任务、MCP 清理、停机语义
- `07-artifact-generation-and-exec-closure.md`：产物落盘、`exec` 验证、错误驱动修复闭环

## 产出数量

- 预计 7 篇，约覆盖 `loop.py` 的全部关键路径

## 分析抓手

- `run` / `_dispatch` / `_process_message`
- `_run_agent_loop` 的迭代退出条件
- `_save_turn` 的会话写入与截断策略
