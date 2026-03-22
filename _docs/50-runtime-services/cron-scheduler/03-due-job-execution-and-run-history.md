# 到期执行与运行历史

## 起点与终点

- 起点：timer tick 触发 `_on_timer`
- 终点：任务状态落盘，next run 更新并重新 arm timer

## 到期判定

`_on_timer` 在当前时刻筛选“enabled 且已到 next_run_at_ms”的任务：

- 批量筛出 due jobs
- 逐个调用 `_execute_job`
- 统一保存 store
- 重新 arm 下一次唤醒

源码锚点：

- 到期筛选与执行：[service.py:L247-L263](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L247-L263)

## 单任务执行状态机

`_execute_job` 的状态更新顺序是固定的：

1. 调用 `on_job(job)` 执行业务逻辑
2. 写 `last_status/last_error`
3. 写 `last_run_at_ms/updated_at_ms`
4. 追加 `run_history` 记录并做长度截断

源码锚点：

- 执行与状态写入：[service.py:L265-L294](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L265-L294)

## 一次性与循环任务差异

- 一次性（`at`）：
  - `delete_after_run=True` 时直接删除 job
  - 否则禁用并清空 next run
- 循环任务（`every/cron`）：
  - 每次完成后即时重算下一次触发时间

源码锚点：

- 后置分支：[service.py:L295-L305](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L295-L305)

## 手动执行与启停行为

- `run_job` 支持手动触发，可通过 `force` 绕过 disabled 限制
- `enable_job` 会同步重算/清空 next run
- `stop` 会取消 timer task，保证不再新触发

源码锚点：

- 手动执行：[service.py:L384-L395](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L384-L395)
- 启停与启用状态：[service.py:L204-L209](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L204-L209)
- 启用/禁用：[service.py:L368-L382](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L368-L382)
