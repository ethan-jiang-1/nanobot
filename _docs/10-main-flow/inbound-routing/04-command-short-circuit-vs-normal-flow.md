# 命令短路与常规路径对照

## 起点与终点

- 起点：`run` 从 inbound 取到一条消息
- 终点：该消息要么立即命令处理，要么进入 `_dispatch -> _process_message`

## 一级短路（run 层）

`/stop`、`/restart`、`/status` 在 `run` 里直接走 priority dispatch，不进入 `_dispatch`：

- 好处：控制命令延迟低，不被业务长任务阻塞
- 结果：控制面与业务面分离

源码锚点：

- 入口分支：[loop.py:L328-L335](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L328-L335)
- priority 注册（含 `/status`）：[builtin.py:L103-L110](file:///Users/bowhead/nanobot/nanobot/command/builtin.py#L103-L110)

## 二级短路（_process_message 层）

普通消息进入 `_process_message` 后还会再分流：

- `msg.channel == "system"`：走系统消息路径
- 命令文本统一走 `commands.dispatch`
- `/new`：清空会话并异步归档快照
- `/help`：直接返回命令说明
- `/status`：若未在 run 层命中，仍可在 exact 路由返回运行态信息
- 其余：进入完整 LLM/工具闭环

源码锚点：

- `_process_message` 分支：[loop.py:L400-L479](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L400-L479)
- 命令分发调用点：[loop.py:L439-L442](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L439-L442)
- `/new` `/help` `/status` 实现：[builtin.py:L44-L110](file:///Users/bowhead/nanobot/nanobot/command/builtin.py#L44-L110)

## 用户可见差异

- `/help` 立即返回文本，不触发模型调用
- `/new` 返回 `New session started.`，并在后台归档旧片段
- system 消息默认回包 `Background task completed.`（若模型无内容）

## 相关测试

- `/help` 包含 `/restart`：[test_restart_command.py:L81-L88](file:///Users/bowhead/nanobot/tests/test_restart_command.py#L81-L88)
- `_dispatch` 统一回包与异常兜底：[test_task_cancel.py:L101-L112](file:///Users/bowhead/nanobot/tests/test_task_cancel.py#L101-L112)
