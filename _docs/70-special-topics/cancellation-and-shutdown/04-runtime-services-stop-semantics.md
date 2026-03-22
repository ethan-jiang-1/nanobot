# 运行时服务的停止语义

## 起点与终点

- 起点：系统进入停机或收到显式停止信号
- 终点：cron、heartbeat、channel 都回到可预测静止态

## cron：停表 + 取消定时器

`CronService.stop` 的核心是停止运行标志并取消活跃 timer/task，避免新任务继续触发。

源码锚点：

- 服务实现：[service.py](file:///Users/bowhead/nanobot/nanobot/cron/service.py)

## heartbeat：停止循环任务

`HeartbeatService.stop` 会关闭 running 状态并取消 loop task，确保不再进入下一轮 tick。

源码锚点：

- 心跳服务：[service.py](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py)

## channels：先停分发后停连接

channel 管理器先停止 outbound 分发，再停各渠道连接，避免“发送半途断链”的乱序行为。

源码锚点：

- channel manager：[manager.py](file:///Users/bowhead/nanobot/nanobot/channels/manager.py)

## 生产建议

- 保持固定停机顺序，不要随意调整
- 对停机过程增加关键日志，便于定位“未完全停止”问题
