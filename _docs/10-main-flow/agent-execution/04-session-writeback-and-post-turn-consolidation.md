# 回合写回与后置归纳

## 起点与终点

- 起点：`_run_agent_loop` 返回 `all_msgs`
- 终点：session 持久化完成，并调度下一轮后台归纳

## 回写步骤

常规路径会执行：

1. `_save_turn(session, all_msgs, 1 + len(history))`
2. `sessions.save(session)`
3. `_schedule_background(maybe_consolidate_by_tokens(session))`

源码锚点：

- 常规回写段：[loop.py:L454-L457](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L454-L457)

## `_save_turn` 的卫生策略

- 跳过空 assistant 消息
- 截断超长 tool 结果（16k 上限）
- 剥离 runtime context
- 多模态图片转稳定占位文本

源码锚点：

- `_save_turn`：[loop.py:L468-L503](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L468-L503)

## preflight 与 post-turn 双阶段

`_process_message` 在调用模型前先 `maybe_consolidate_by_tokens`，回合结束后再异步调度一次，形成“前置防爆 + 后置收敛”。

源码锚点：

- preflight：[loop.py:L424-L424](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L424-L424)
- post-turn：[loop.py:L456-L456](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L456-L456)

## 相关测试

- 归纳先于 LLM 调用：[test_loop_consolidation_tokens.py:L155-L190](file:///Users/bowhead/nanobot/tests/test_loop_consolidation_tokens.py#L155-L190)

