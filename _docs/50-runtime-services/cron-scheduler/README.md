# cron-scheduler

本目录聚焦 CronService：任务如何定义、持久化、计算下一次触发并执行。

## 本主题要回答的问题

- jobs.json 如何映射到内存态并保证兼容性
- next run 如何计算，timer 如何重置
- 到期任务如何执行、记录历史并更新状态
- `cron` 工具如何桥接服务并防止递归调度

## 建议文档拆分

- `01-cron-store-model-and-persistence.md`：数据模型、加载保存、外部修改重载
- `02-schedule-computation-and-timer-arm.md`：at/every/cron 计算与定时器唤醒
- `03-due-job-execution-and-run-history.md`：到期执行、成功失败状态、历史截断
- `04-cron-tool-bridge-and-recursion-guard.md`：工具层 add/list/remove 与上下文限制

## 产出数量

- 预计 4 篇，覆盖 `cron/service.py` 与 `agent/tools/cron.py` 的运行关键路径

## 分析抓手

- `CronService.start` / `_arm_timer` / `_on_timer`
- `CronService._execute_job` / `run_job`
- `CronTool.execute`
