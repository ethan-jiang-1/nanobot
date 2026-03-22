# 调度计算与定时器唤醒

## 起点与终点

- 起点：任务被添加/启用，或服务启动后重算 next run
- 终点：`_arm_timer` 安排下一次准确唤醒

## 三种调度语义

`_compute_next_run` 统一处理三类 schedule：

- `at`：只在未来时间点触发，过期即返回 `None`
- `every`：以当前时间为基准推导 `now + interval`
- `cron`：用 `croniter + ZoneInfo` 计算下一个触发点

源码锚点：

- 计算函数：[service.py:L20-L46](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L20-L46)

## 入库前校验

`_validate_schedule_for_add` 做关键约束：

- `tz` 只能和 `cron` 搭配
- 时区名必须可被 `ZoneInfo` 解析

这样把“不可运行任务”挡在入口层。

源码锚点：

- 参数校验：[service.py:L49-L60](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L49-L60)

## 启动阶段重算

`start` 不直接相信磁盘里的 next run，而是重算并重新保存：

1. 加载 store
2. `_recompute_next_runs` 批量刷新 enabled job 的 next run
3. 保存并 `arm` 下一次唤醒

源码锚点：

- 启动流程：[service.py:L195-L203](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L195-L203)
- 重算逻辑：[service.py:L211-L219](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L211-L219)

## timer 重置机制

`_arm_timer` 采用“单定时器重置”策略：

- 每次先取消旧 timer
- 取全局最早 `next_run_at_ms`
- 只创建一个 `asyncio.create_task(tick())`
- tick 到点后进入 `_on_timer`，执行后再次 `arm`

这避免了“每个 job 一个 sleep task”的任务风暴。

源码锚点：

- 取最早唤醒点：[service.py:L220-L226](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L220-L226)
- arm 与 tick：[service.py:L228-L246](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L228-L246)
- timer 到点回调：[service.py:L247-L263](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L247-L263)
