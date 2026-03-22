# 异常处理与禁用语义

## 起点与终点

- 起点：heartbeat 启动参数 `enabled/interval_s` 生效
- 终点：异常、空任务、停机都能回到可预期状态

## 禁用语义

禁用是“启动期短路”，不是运行中降级：

- `enabled=False` 时 `start()` 直接返回
- 不创建 loop task，不产生任何 tick

源码锚点：

- 禁用短路：[service.py:L111-L116](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L111-L116)
- 配置字段：[schema.py:L85-L97](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L85-L97)

## 运行中错误隔离

错误被分成两层隔离：

- `_run_loop` 层：单轮异常只记日志，不退出循环
- `_tick` 层：执行期异常被捕获并打印 exception

这保证一次失败不会导致整个心跳服务永久停止。

源码锚点：

- loop 容错：[service.py:L131-L142](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L131-L142)
- tick 容错：[service.py:L174-L175](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L174-L175)

## 结果静默策略

即使 `action=run`，也不一定通知外部渠道：

- 当 `on_execute` 返回空响应时，不通知
- 后评估判定 `should_notify=False` 时，显式静默
- 无外部会话目标时，notify 回调在 `cli` 直接返回

源码锚点：

- post-run 评估与静默：[service.py:L165-L173](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L165-L173)
- cli 目标静默：[commands.py:L631-L633](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L631-L633)

## 手动触发边界

`trigger_now` 用于临时触发，不改变循环状态：

- 仍走 `_decide` 决策
- 非 `run` 或无 `on_execute` 时返回 `None`
- 仅返回执行结果，不负责对外通知

源码锚点：

- 手动触发：[service.py:L177-L185](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L177-L185)
