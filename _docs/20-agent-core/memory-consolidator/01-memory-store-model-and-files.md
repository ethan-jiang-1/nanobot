# MemoryStore 数据模型与文件职责

## 目标

这篇先讲清 memory 子系统最核心的数据落点：`MEMORY.md` 与 `HISTORY.md`。

## 双层文件模型

`MemoryStore` 是两层结构：

- `MEMORY.md`：长期事实集合，面向“当前状态”
- `HISTORY.md`：按时间追加的历史条目，面向“可回溯与可 grep”

源码锚点：

- 类定义与路径初始化：[memory.py:L75-L84](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L75-L84)

## 读写接口分工

- `read_long_term`：读取 `MEMORY.md` 当前内容
- `write_long_term`：覆盖写 `MEMORY.md`
- `append_history`：向 `HISTORY.md` 追加段落
- `get_memory_context`：给 ContextBuilder 提供可注入文本

源码锚点：

- [memory.py:L86-L100](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L86-L100)

## 为什么是“覆盖 + 追加”

- 长期记忆强调“最新一致视图”，所以 `MEMORY.md` 覆盖写
- 历史强调“事件序列完整性”，所以 `HISTORY.md` 仅追加

这让模型下一轮读取时既有可直接使用的最新事实，也保留审计轨迹。

## 与 ContextBuilder 的连接

ContextBuilder 在 system prompt 中只读取 memory context，不读取 history。

意味着：

- `MEMORY.md` 直接影响后续推理
- `HISTORY.md` 更多用于归档与检索分析

关联实现：

- memory context 注入：[context.py:L35-L38](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L35-L38)
- prompt 组装：[context.py:L27-L54](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L27-L54)

## 历史格式

`_format_messages` 会把消息规约为可读行：

- 时间前缀（精确到分钟）
- 角色名大写
- 可选 `tools_used`
- 文本 content

源码锚点：

- [memory.py:L102-L112](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L102-L112)
