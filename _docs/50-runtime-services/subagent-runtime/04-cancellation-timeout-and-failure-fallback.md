# 取消、超时与失败兜底

## 起点与终点

- 起点：后台子任务正在执行，收到 `/stop` 或运行异常
- 终点：任务被回收、状态可解释、主流程不被拖垮

## 会话级取消

主循环收到 `/stop` 后会同步取消两类任务：

- 前台 `_active_tasks[session_key]`
- 该会话关联的子代理任务 `subagents.cancel_by_session(session_key)`

对用户返回统一统计文案：`Stopped N task(s).`

源码锚点：

- stop 处理：[loop.py:L288-L302](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L288-L302)
- 子代理会话取消：[subagent.py:L223-L231](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L223-L231)

## 任务索引与自动清理

`SubagentManager` 用两张表保证可管理性：

- `_running_tasks`: task_id -> asyncio.Task
- `_session_tasks`: session_key -> set(task_id)

并通过 done callback 自动清理索引，避免泄漏。

源码锚点：

- 索引初始化：[subagent.py:L47-L49](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L47-L49)
- 回调清理：[subagent.py:L70-L77](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L70-L77)

## 失败兜底语义

`_run_subagent` 捕获异常后不会中断主代理，而是：

- 把异常转成 `Error: ...` 文本
- 仍走 `_announce_result(..., status="error")` 回注

这保证用户可见“任务失败”而不是静默丢失。

源码锚点：

- 异常分支：[subagent.py:L163-L167](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L163-L167)

## 超时边界

子代理没有独立全局超时参数，主要边界来自两处：

- 迭代上限 `max_iterations = 15`
- `exec` 工具自身 timeout（由 `ExecToolConfig` 注入）

源码锚点：

- 迭代上限：[subagent.py:L116-L123](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L116-L123)
- exec timeout 注入：[subagent.py:L101-L104](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L101-L104)
