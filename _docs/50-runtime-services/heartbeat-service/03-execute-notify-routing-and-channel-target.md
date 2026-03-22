# 执行与通知路由

## 起点与终点

- 起点：heartbeat 决策返回 `action=run`
- 终点：任务结果进入 agent 主链路，并按目标渠道决定是否对外通知

## 执行回调桥接

gateway 注入的 `on_heartbeat_execute` 不直接调用 provider，而是走完整 agent 流程：

- 先用 `_pick_heartbeat_target` 选择可投递目标
- 再 `agent.process_direct(tasks, session_key="heartbeat", ...)`
- 进度回调置空，避免后台心跳刷屏

源码锚点：

- 目标选择：[commands.py:L595-L610](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L595-L610)
- execute 回调：[commands.py:L611-L625](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L611-L625)

## 通知回调桥接

`on_heartbeat_notify` 的行为是“仅外部渠道投递”：

- 再次选目标 channel/chat_id
- 若目标是 `cli` 则静默返回
- 否则发布 outbound 交给 channel 层发送

源码锚点：

- notify 回调：[commands.py:L627-L633](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L627-L633)

## Service 内部执行路径

`_tick` 在 `run` 分支中的关键链路：

1. 调用 `on_execute(tasks)` 获取 response
2. 对 response 做后评估（是否值得通知）
3. 评估通过才调用 `on_notify(response)`

源码锚点：

- run 分支与回调：[service.py:L155-L173](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L155-L173)

## 与普通消息路径的关系

heartbeat 执行最终落到 `AgentLoop._process_message`，所以：

- 共享同一工具体系和上下文注入
- 共享同一 session 管理与 memory consolidation
- 与普通用户消息差异仅在“触发源”和“回调路由”

源码锚点：

- direct 入口：[loop.py:L505-L517](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L505-L517)
