# cancellation-and-shutdown

本目录聚焦“系统如何停得下来且停得干净”：命令取消、后台任务回收、服务停机顺序。

## 本主题要回答的问题

- `/stop` 到底取消了哪些任务，哪些不会被误伤
- 子代理任务如何索引、如何避免泄漏
- gateway 退出时为什么要按固定顺序关闭组件
- cron/heartbeat/channel 在停机中各自保证什么语义

## 建议文档拆分

- `01-command-stop-and-session-task-cancel.md`：会话级取消语义
- `02-subagent-task-indexing-and-cleanup.md`：后台任务索引与回收
- `03-gateway-shutdown-order-and-resource-drain.md`：关停顺序与资源 drain
- `04-runtime-services-stop-semantics.md`：cron/heartbeat/channel 停止语义

## 分析抓手

- `AgentLoop._handle_stop`
- `SubagentManager.cancel_by_session`
- `gateway` finally 关闭顺序
- `CronService.stop` / `HeartbeatService.stop` / `ChannelManager.stop_all`
