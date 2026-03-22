# `/stop` 与会话级取消

## 起点与终点

- 起点：用户在当前会话发送 `/stop`
- 终点：当前会话相关任务被取消，并返回可见统计结果

## 取消范围

`/stop` 不是全局 kill，而是“当前 session key 精准取消”：

- 取消 `_active_tasks[session_key]` 的前台处理任务
- 同步触发子代理会话任务取消
- 不影响其他会话的并发处理

源码锚点：

- stop 处理：[loop.py](file:///Users/bowhead/nanobot/nanobot/agent/loop.py)
- 子代理取消入口：[subagent.py](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py)

## 用户可见语义

- 返回 `Stopped N task(s).`
- 若没有可取消任务，也保持可解释的空结果

## 设计价值

- 保证多会话并发下的隔离性
- 避免“一个 stop 把全局任务都杀掉”的事故
- 为生产排障提供可量化反馈
