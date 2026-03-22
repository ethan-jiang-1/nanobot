# subagent-runtime

本目录聚焦后台子任务执行：主代理如何把任务拆出、子代理如何运行、结果如何回传。

## 本主题要回答的问题

- `spawn` 如何携带上下文并安全地下发任务
- 子代理循环与主代理循环在工具能力上如何隔离
- 后台结果如何通过 system 通道回注主链路
- 超时、取消与失败时如何保证可观测与可恢复

## 建议文档拆分

- `01-spawn-tool-context-and-task-hand-off.md`：参数契约、来源上下文、任务创建
- `02-subagent-loop-and-tool-sandbox.md`：子代理消息循环与可用工具边界
- `03-system-channel-callback-and-result-delivery.md`：完成回调与 system 消息回注路径
- `04-cancellation-timeout-and-failure-fallback.md`：取消、异常、兜底回复策略

## 产出数量

- 预计 4 篇，覆盖 `agent/tools/spawn.py` 与 `agent/subagent.py` 核心机制

## 分析抓手

- `SpawnTool.execute`
- `SubagentManager.spawn` / `_run_subagent`
- `AgentLoop._process_message` 的 system 分支
