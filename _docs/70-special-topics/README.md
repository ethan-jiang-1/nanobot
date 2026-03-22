# 70-专题

本目录聚焦跨模块机制，不再按单一链路拆分，而是按“线上最常见问题类型”组织。

## 选题原则

- 只收纳横跨多个模块、单看某一目录无法讲清的问题
- 优先覆盖线上高频风险：副作用边界、停机与取消、一致性退化
- 每个子目录都要给出“故障现象 -> 定位抓手 -> 关键源码”

## 子目录

- `tool-safety-boundaries/`：工具能力与副作用边界
- `cancellation-and-shutdown/`：取消语义、后台任务回收、停机顺序
- `history-and-memory-consistency/`：会话历史卫生、归纳边界与恢复语义

## 阅读路径

- 先看 `tool-safety-boundaries/`，建立“什么能做、做到哪一步算安全”
- 再看 `cancellation-and-shutdown/`，理解“中断与退出时如何不丢状态”
- 最后看 `history-and-memory-consistency/`，理解“长期运行后如何保持一致性”

## 故障定位速查

| 现象 | 核心文件 | 关键函数 |
|---|---|---|
| 工具执行后出现越权或副作用失控 | [shell.py](../../nanobot/agent/tools/shell.py), [web.py](../../nanobot/agent/tools/web.py), [cron.py](../../nanobot/agent/tools/cron.py) | [ExecTool.execute](../../nanobot/agent/tools/shell.py), [WebFetchTool.execute](../../nanobot/agent/tools/web.py), [CronTool.execute](../../nanobot/agent/tools/cron.py) |
| `/stop` 后仍有后台任务继续跑 | [loop.py](../../nanobot/agent/loop.py), [subagent.py](../../nanobot/agent/subagent.py) | [_handle_stop](../../nanobot/agent/loop.py), [cancel_by_session](../../nanobot/agent/subagent.py) |
| 历史越来越乱、模型上下文失真 | [loop.py](../../nanobot/agent/loop.py), [manager.py](../../nanobot/session/manager.py), [memory.py](../../nanobot/agent/memory.py) | [_save_turn](../../nanobot/agent/loop.py), [Session.get_history](../../nanobot/session/manager.py), [maybe_consolidate_by_tokens](../../nanobot/agent/memory.py) |
