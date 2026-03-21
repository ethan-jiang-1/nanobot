# save_memory 工具协议与参数归一化

## 目标

这篇拆解 `MemoryStore.consolidate` 的协议层：它如何要求模型产出、如何兼容不同 provider 的 tool 参数形态。

## 协议定义

memory 归纳通过单工具 `save_memory` 完成，要求两个字段：

- `history_entry`
- `memory_update`

源码锚点：

- `_SAVE_MEMORY_TOOL`：[memory.py:L21-L45](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L21-L45)

## 调用策略

默认先强制 `tool_choice={"name":"save_memory"}`，要求模型必须走工具调用路径。  
若 provider 返回 “tool_choice 不支持” 错误，再回退 `tool_choice="auto"` 重试一次。

源码锚点：

- 主调用与回退：[memory.py:L138-L156](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L138-L156)
- 错误识别标记：[memory.py:L61-L72](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L61-L72)

相关测试：

- fallback 成功：[test_memory_consolidation_types.py:L379-L410](file:///Users/bowhead/nanobot/tests/test_memory_consolidation_types.py#L379-L410)
- fallback 后仍无 tool call：[test_memory_consolidation_types.py:L412-L434](file:///Users/bowhead/nanobot/tests/test_memory_consolidation_types.py#L412-L434)

## 参数归一化

provider 返回参数形态可能是：

- dict
- JSON 字符串
- list[dict]

`_normalize_save_memory_args` 统一把它们规约成 dict，不合法则返回 `None`。

源码锚点：

- [memory.py:L53-L59](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L53-L59)

相关测试：

- 字符串 JSON 参数：[test_memory_consolidation_types.py:L110-L136](file:///Users/bowhead/nanobot/tests/test_memory_consolidation_types.py#L110-L136)
- list 参数提取首个 dict：[test_memory_consolidation_types.py:L167-L194](file:///Users/bowhead/nanobot/tests/test_memory_consolidation_types.py#L167-L194)
- list 非法内容拒绝：[test_memory_consolidation_types.py:L196-L242](file:///Users/bowhead/nanobot/tests/test_memory_consolidation_types.py#L196-L242)

## 文本归一化

`_ensure_text` 把非字符串值序列化为 JSON 文本，确保落盘时不会抛类型异常。

源码锚点：

- [memory.py:L48-L50](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L48-L50)

相关测试：

- dict 参数可写入：[test_memory_consolidation_types.py:L84-L108](file:///Users/bowhead/nanobot/tests/test_memory_consolidation_types.py#L84-L108)

## 完整性校验

在真正写文件前会严格校验：

- 必须存在两个 required 字段
- 字段值不可为 null
- `history_entry` 归一化后不可为空

任一失败都会进入失败路径，不写入部分结果。

相关测试：

- 缺字段/空字段保护：[test_memory_consolidation_types.py:L244-L329](file:///Users/bowhead/nanobot/tests/test_memory_consolidation_types.py#L244-L329)
