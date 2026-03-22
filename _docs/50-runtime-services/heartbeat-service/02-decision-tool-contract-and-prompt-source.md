# 决策契约与输入来源

## 起点与终点

- 起点：`_tick` 读取 `HEARTBEAT.md`
- 终点：`_decide` 返回标准化 `(action, tasks)`

## 输入来源

Heartbeat 不读取会话历史，而是读取 workspace 下单文件：

- 路径固定为 `workspace / "HEARTBEAT.md"`
- 文件缺失或读取失败会直接跳过本轮

这使心跳决策可被文件系统显式驱动。

源码锚点：

- 文件路径与读取：[service.py:L73-L84](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L73-L84)
- 空内容短路：[service.py:L147-L150](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L147-L150)

## 虚拟工具契约

`_HEARTBEAT_TOOL` 强制模型用 tool call 给出结构化决策：

- `action`: `skip | run`
- `tasks`: run 时的自然语言任务摘要

模型若不返回 tool call，系统默认当作 `skip`。

源码锚点：

- 工具 schema：[service.py:L14-L37](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L14-L37)
- 默认 skip 逻辑：[service.py:L105-L109](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L105-L109)

## Prompt 组成

`_decide` 的输入由两段组成：

- system：要求“必须调用 heartbeat 工具汇报”
- user：当前时间 + HEARTBEAT.md 原文

这避免了自由文本解析造成的不稳定判定。

源码锚点：

- `_decide` 请求组包：[service.py:L85-L103](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L85-L103)

## 与写作规范的衔接

模板已经把 HEARTBEAT.md 作为周期任务入口：

- 新增周期任务：编辑 HEARTBEAT.md
- 完成后移除：编辑 HEARTBEAT.md
- 不建议把周期任务写到 MEMORY.md

源码锚点：

- 规则提示：[AGENTS.md:L12-L21](file:///Users/bowhead/nanobot/nanobot/templates/AGENTS.md#L12-L21)
