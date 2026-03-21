# Context 构建与 runtime 元数据注入

## 起点与终点

- 起点：`_process_message` 准备调用 `context.build_messages`
- 终点：生成可直接送入 provider 的 message 列表

## 组包结构

`build_messages` 输出固定结构：

- 首条 system：`build_system_prompt(...)`
- 中间历史：`history`
- 末条当前消息：`current_role + merged content`

源码锚点：

- `build_messages`：[context.py:L120-L145](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L120-L145)

## runtime 注入策略

runtime 元数据通过 `_build_runtime_context(channel, chat_id)` 生成，并与当前用户输入合并为同一条消息：

- 文本：`runtime + "\n\n" + user_text`
- 多模态：在列表首部插入 `{"type":"text","text":runtime}`

这样避免连续同角色消息触发 provider 拒绝。

源码锚点：

- runtime 构建：[context.py:L100-L106](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L100-L106)
- 合并逻辑：[context.py:L131-L140](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L131-L140)

## 与持久化的配合

runtime 注入只用于本轮推理，`_save_turn` 会在写 session 前剥离，避免污染长期历史。

源码锚点：

- 剥离逻辑：[loop.py:L479-L500](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L479-L500)

