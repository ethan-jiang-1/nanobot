# 历史切片与工具调用边界修复

## 目标

这篇解释 `Session.get_history` 为何不只是“取最后 N 条”，而是必须修复 tool_call 合法性边界。

## 切片主流程

`get_history(max_messages)` 的处理链：

1. 先跳过已归纳前缀 `messages[last_consolidated:]`
2. 再截取窗口 `[-max_messages:]`
3. 尽量把开头对齐到第一个 user
4. 用 `_find_legal_start` 去掉 orphan tool 结果前缀
5. 最后只保留模型必要字段输出

源码锚点：

- [manager.py:L69-L93](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L69-L93)

## orphan tool result 问题

固定窗口切片时，可能把 assistant 的 `tool_calls` 裁掉，却保留后续 `role=tool` 消息。  
部分 provider 会直接拒绝这种“无声明的 tool_call_id”。

`_find_legal_start` 会线性扫描并维护 declared tool_call_id 集，若发现 orphan，就推进 start 并重建声明集。

源码锚点：

- [manager.py:L46-L67](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L46-L67)

## 为什么先“找 user”再“找 legal start”

- 先找 user：尽量避免从 assistant/tool 中间开头
- 再 legal 修复：兜底处理仍可能出现的 orphan

两层策略组合后，兼顾了语义完整性与 provider 结构合法性。

## 输出字段最小化

输出 history 时不会原样透传所有字段，只保留：

- `role`
- `content`
- 可选 `tool_calls` / `tool_call_id` / `name`

减少无关字段干扰模型输入。

源码锚点：

- [manager.py:L86-L92](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L86-L92)

## 测试覆盖

- 回归：窗口裁切导致 orphan 时自动修复  
  [test_session_manager_history.py:L36-L48](file:///Users/bowhead/nanobot/tests/test_session_manager_history.py#L36-L48)
- 正例：合法 tool 对完整保留  
  [test_session_manager_history.py:L52-L65](file:///Users/bowhead/nanobot/tests/test_session_manager_history.py#L52-L65)
- 边界：全 orphan 前缀、mid-group 裁切、空会话  
  [test_session_manager_history.py:L102-L122](file:///Users/bowhead/nanobot/tests/test_session_manager_history.py#L102-L122)  
  [test_session_manager_history.py:L126-L146](file:///Users/bowhead/nanobot/tests/test_session_manager_history.py#L126-L146)
