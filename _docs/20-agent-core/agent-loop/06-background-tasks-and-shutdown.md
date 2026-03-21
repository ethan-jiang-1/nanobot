# 后台任务与停机语义

## 目标

这篇聚焦 AgentLoop 如何处理“非阻塞归档任务”和“优雅关闭”。

## 为什么需要后台归档

在 `/new` 或“开启自动归档”的场景里，部分历史会异步交给记忆模块处理。  
如果同步执行，会显著增加用户感知延迟。

所以 loop 采用后台任务模型：主链路先响应，归档稍后完成。

## ` _schedule_memory_archive` 的行为

该方法会创建后台任务并纳入 `_background_tasks` 集合管理：

- 任务开始时复制消息快照，避免后续会话改写影响归档输入
- 任务结束自动从集合移除
- 任务异常只记录日志，不中断主链路

源码锚点：

- 调度函数：[loop.py:L362-L369](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L362-L369)
- 实际归档函数：[loop.py:L579-L592](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L579-L592)

## 优雅关闭策略

`close_mcp` 会先等待后台任务：

1. `await gather(*background_tasks, return_exceptions=True)`
2. 清空 `_background_tasks`
3. `await tools.close_mcp()`

这三步确保：

- 归档尽量完成
- 资源回收顺序可预测
- 即便个别后台任务失败，也不会卡死 shutdown

源码锚点：

- `close_mcp`：[loop.py:L340-L350](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L340-L350)

## 与 `stop` 的区别

- `stop`：仅停止消费新消息
- `close_mcp`：执行最终清理

这两个动作分离，适合“先停流量，再做收尾”的实际部署场景。

## 常见风险与防线

- 风险：进程突然退出导致后台任务丢失  
  防线：关键路径可在需要时改成同步归档
- 风险：后台任务异常影响主流程  
  防线：异常捕获+日志，不向主链路抛出
- 风险：关闭时仍有任务进行中  
  防线：shutdown 前统一等待任务完成
