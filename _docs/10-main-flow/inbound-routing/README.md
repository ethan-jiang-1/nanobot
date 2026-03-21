# inbound-routing

本目录聚焦“消息如何进入核心”：从平台回调，到标准化 `InboundMessage`，再到 AgentLoop 分发。

## 本主题要回答的问题

- channel 层如何把平台事件标准化成统一入站事件
- session key 如何派生，何时使用 override
- `run` 主循环如何命令短路与任务分发
- system 消息如何回注主链路

## 建议文档拆分

- `01-channel-message-to-inbound-event.md`：BaseChannel `_handle_message` 抽象
- `02-inbound-event-model-and-session-key.md`：InboundMessage/OutboundMessage 数据模型
- `03-agent-run-loop-dispatch.md`：run 循环、`/stop`/`/restart` 拦截、任务登记
- `04-command-short-circuit-vs-normal-flow.md`：命令路径与常规路径差异

## 产出数量

- 预计 4 篇，覆盖“平台输入 -> Agent 入口”链路

