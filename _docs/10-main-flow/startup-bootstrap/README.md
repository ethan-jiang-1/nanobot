# startup-bootstrap

本目录拆解主链路启动阶段：进程如何装配核心对象、如何并发拉起服务、以及如何优雅收尾。

## 本主题要回答的问题

- gateway 启动时按什么顺序创建 bus/provider/agent/channels
- cron 与 heartbeat 为什么要在 agent 创建后再绑定回调
- `agent.run` 与 `channels.start_all` 如何并发运行
- 异常或退出时如何保证资源按顺序关闭

## 建议文档拆分

- `01-cli-gateway-bootstrap.md`：gateway 入口、配置解析、运行态创建
- `02-provider-bus-agent-wiring.md`：MessageBus/Provider/AgentLoop 装配关系
- `03-channel-manager-startup.md`：ChannelManager 初始化与启动语义
- `04-shutdown-sequence-and-resource-drain.md`：关闭顺序、后台任务 drain、资源回收

## 产出数量

- 预计 4 篇，覆盖 `commands.py` gateway 主流程与协同对象

