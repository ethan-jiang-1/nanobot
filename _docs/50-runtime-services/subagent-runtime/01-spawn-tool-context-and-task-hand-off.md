# Spawn 上下文与任务下发

## 起点与终点

- 起点：主代理调用 `spawn(task=...)`
- 终点：任务被 `SubagentManager` 接管并进入后台执行

## SpawnTool 的职责边界

`SpawnTool` 本身只做三件事：

- 暴露 `spawn` 参数契约（`task` 必填，`label` 可选）
- 保存来源上下文（channel/chat_id/session_key）
- 把任务转交给 manager，不直接执行任务

源码锚点：

- 工具定义与执行：[spawn.py:L11-L65](file:///Users/bowhead/nanobot/nanobot/agent/tools/spawn.py#L11-L65)

## 上下文注入来源

来源上下文不是用户显式传入，而是每轮由 AgentLoop 注入：

- `_set_tool_context` 会对 `spawn` 调用 `set_context`
- 因此 spawn 生成的后台任务天然知道“该回哪个会话”

源码锚点：

- 工具上下文注入：[loop.py:L159-L165](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L159-L165)

## hand-off 细节

`SubagentManager.spawn` 负责真正接管：

- 生成 `task_id` 与展示 label
- 构造 origin（channel/chat_id）
- `create_task(_run_subagent(...))` 异步启动
- 把 task_id 记录到 `_running_tasks` 与 `_session_tasks`

源码锚点：

- spawn hand-off：[subagent.py:L50-L80](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L50-L80)

## 用户可见回执

主代理在当前回合立即返回“已启动”文本，而不是等待后台完成：

- 保持前台回合低延迟
- 后续结果通过 system 入站异步回注

源码锚点：

- 即时回执：[subagent.py:L79-L80](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L79-L80)
