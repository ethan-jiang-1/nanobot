# 子代理任务索引与清理闭环

## 起点与终点

- 起点：`spawn` 创建后台子任务
- 终点：任务完成/失败/取消后索引被回收，不留悬挂状态

## 两张关键索引表

`SubagentManager` 用双索引实现“可定位 + 可批量取消”：

- `_running_tasks`: `task_id -> asyncio.Task`
- `_session_tasks`: `session_key -> set(task_id)`

源码锚点：

- 管理器实现：[subagent.py](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py)

## 自动回收机制

- 创建任务后绑定 done callback
- callback 内从两张表同步移除记录
- 会话任务集为空时可进一步清理 session 索引

## 失败处理

- `_run_subagent` 捕获异常并回注 error 文本
- 保证“任务失败可见”，而不是静默丢失

源码锚点：

- 失败兜底与回注：[subagent.py](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py)
