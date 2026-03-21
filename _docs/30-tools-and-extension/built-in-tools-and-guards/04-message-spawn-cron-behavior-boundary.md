# message / spawn / cron 的行为边界

## 起点与终点

- 起点：模型调用“执行型工具”向外部世界施加动作
- 终点：动作被投递、后台执行或调度，并保持会话边界可控

## message：主动投递到会话

`message` 可以显式发送到指定 `channel/chat_id`，也可依赖回合上下文默认值。  
当发送到当前默认目标成功，会标记“本回合已主动发送”。

源码锚点：

- 参数与执行：[message.py:L48-L109](file:///Users/bowhead/nanobot/nanobot/agent/tools/message.py#L48-L109)

## spawn：把任务移交子代理

`spawn` 将任务下发给 `SubagentManager`，并携带来源 `channel/chat_id/session_key`，用于后续回传。

源码锚点：

- spawn 工具实现：[spawn.py:L11-L65](file:///Users/bowhead/nanobot/nanobot/agent/tools/spawn.py#L11-L65)
- 子代理工具集与循环：[subagent.py:L82-L148](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L82-L148)

## cron：延迟与周期任务

`cron` 支持 `add/list/remove`，并要求当前会话上下文存在；同时禁止在 cron 回调内部再次创建新任务，避免递归调度。

源码锚点：

- cron 主逻辑：[cron.py:L74-L146](file:///Users/bowhead/nanobot/nanobot/agent/tools/cron.py#L74-L146)
- list/remove 与状态格式化：[cron.py:L147-L199](file:///Users/bowhead/nanobot/nanobot/agent/tools/cron.py#L147-L199)

## 共同边界

- 三者都依赖 `AgentLoop._set_tool_context` 注入路由上下文
- 都属于“副作用工具”，结果不只影响模型上下文，还影响外部系统行为
