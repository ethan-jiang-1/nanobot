# tool-registry-and-runtime

本目录聚焦工具体系的运行时主干：Tool 抽象、ToolRegistry 管理、AgentLoop 的模型-工具迭代闭环。

## 本主题要回答的问题

- 一个工具最小需要实现什么契约
- 工具参数如何 cast/validate，再进入执行
- AgentLoop 如何在多轮中持续执行工具直到收敛
- 工具执行失败如何反馈给模型进行下一轮修正

## 建议文档拆分

- `01-tool-abstraction-and-schema-contract.md`
- `02-toolregistry-registration-and-dispatch.md`
- `03-agentloop-tool-execution-iteration.md`
- `04-tool-context-injection-and-turn-semantics.md`
- `05-why-llm-decides-to-write-files-or-run-commands.md`
