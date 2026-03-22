# 工具能力面与注册闸门

## 起点与终点

- 起点：`AgentLoop` 初始化并注册默认工具
- 终点：形成“可用工具集合”，并把受限能力按配置开关显式控制

## 能力面分层

工具大致分三类：

- 只读/低副作用：`list_dir`、`read_file`
- 可写/强副作用：`write_file`、`edit_file`、`exec`
- 外部系统交互：`web_search`、`web_fetch`、`message`、`spawn`、`cron`

源码锚点：

- 工具注册总入口：[loop.py](file:///Users/bowhead/nanobot/nanobot/agent/loop.py)
- 工具基类与 schema：[base.py](file:///Users/bowhead/nanobot/nanobot/agent/tools/base.py)

## 注册闸门

并非所有能力都无条件开启：

- `exec` 受 `config.tools.exec.enable` 控制
- `web_search` 受 `config.tools.web_search.enable` 控制
- `cron` 依赖 gateway/agent 注入的 `CronService`

源码锚点：

- exec/web_search 注册分支：[loop.py](file:///Users/bowhead/nanobot/nanobot/agent/loop.py)
- 工具配置模型：[schema.py](file:///Users/bowhead/nanobot/nanobot/config/schema.py)

## 生产建议

- 默认按“最小能力集”启动，再逐步开启高副作用工具
- 把“能力开关 + 风险说明”作为变更评审必填项
- 新工具上线前，先补失败语义与回归测试
