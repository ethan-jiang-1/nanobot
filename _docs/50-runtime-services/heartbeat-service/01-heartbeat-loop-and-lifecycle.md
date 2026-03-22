# 心跳循环与生命周期

## 起点与终点

- 起点：gateway 创建 HeartbeatService 并注入 execute/notify 回调
- 终点：服务按 interval 周期 tick，停机时可安全停止

## 装配入口

HeartbeatService 在 gateway 中用配置项初始化：

- workspace：用于定位 `HEARTBEAT.md`
- provider/model：用于“是否执行”决策
- `on_execute` / `on_notify`：执行与投递桥接
- `interval_s` / `enabled`：控制节奏与开关

源码锚点：

- 创建与注入：[commands.py:L611-L644](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L611-L644)
- 配置来源：[schema.py:L85-L97](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L85-L97)

## 启停语义

- `start`：disabled 时直接返回；已运行时防重入
- `start` 成功后创建 `_run_loop` 后台 task
- `stop`：标记 `_running=False` 并取消 task

源码锚点：

- 启动逻辑：[service.py:L111-L123](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L111-L123)
- 停止逻辑：[service.py:L124-L130](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L124-L130)

## 周期循环

`_run_loop` 的核心是“sleep -> tick”：

- 每轮先 `sleep(interval_s)`
- 醒来后再次确认 `_running`
- 执行 `_tick`
- 捕获 `CancelledError` 退出，其他异常仅记录并继续下一轮

源码锚点：

- 主循环：[service.py:L131-L142](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L131-L142)

## 与主进程关闭协同

gateway 的关闭顺序中，heartbeat 先于 agent stop：

1. `await agent.close_mcp()`
2. `heartbeat.stop()`
3. `cron.stop()`
4. `agent.stop()`

这样可以减少停机阶段新 heartbeat tick 的竞争。

源码锚点：

- 关闭顺序：[commands.py:L671-L676](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L671-L676)
