# heartbeat-service

本目录聚焦 HeartbeatService：周期 tick、模型决策、触发执行与通知发送。

## 本主题要回答的问题

- 心跳循环如何启动、停止与防重入
- heartbeat 虚拟工具契约如何约束模型输出
- `run` 决策后如何接入 agent 执行链并路由通知
- 关闭开关、异常与降级策略如何影响运行态

## 建议文档拆分

- `01-heartbeat-loop-and-lifecycle.md`：start/stop、loop、tick 调度节奏
- `02-decision-tool-contract-and-prompt-source.md`：HEARTBEAT.md 输入与 tool call 判定
- `03-execute-notify-routing-and-channel-target.md`：执行回调、通知回调、目标选择
- `04-error-handling-and-disable-semantics.md`：异常恢复、禁用模式、边界行为

## 产出数量

- 预计 4 篇，覆盖 `heartbeat/service.py` 与 gateway 绑定逻辑

## 分析抓手

- `HeartbeatService.start` / `_run_loop` / `_tick`
- `HeartbeatService._decide` / `trigger_now`
- `commands.py` 中 heartbeat callbacks
