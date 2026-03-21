# gateway 启动入口与运行骨架

## 起点与终点

- 起点：用户执行 `nanobot gateway`
- 终点：进入 `asyncio.gather(agent.run(), channels.start_all())` 长驻运行

## 主流程

`gateway` 入口会按固定顺序完成初始化：

1. 读取并归一化配置（含 workspace）
2. 同步 workspace 模板
3. 构建 `MessageBus`、Provider、`SessionManager`
4. 创建 `CronService`、`AgentLoop`
5. 创建 `ChannelManager` 与 `HeartbeatService`
6. 启动 cron + heartbeat + agent + channels

源码锚点：

- gateway 主入口：[commands.py:L494-L678](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L494-L678)

## 关键状态切换

- 配置态：`_load_runtime_config` 之后，`config.workspace_path` 成为后续单一事实来源
- 组件态：`agent` 创建后，cron callback 才能绑定到 `agent.process_direct`
- 运行态：`run()` 内从“初始化”切到“并发长跑”

## 同步/异步边界

- `gateway` 函数本身是同步入口
- `run()` 是异步主协程，由 `asyncio.run` 承载
- agent 与 channels 通过 `gather` 并发执行，不互相阻塞

## 失败路径

- 运行期异常会打印 crash traceback
- `finally` 块始终执行，确保进入关闭序列

源码锚点：

- 运行与异常分支：[commands.py:L657-L678](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L657-L678)

