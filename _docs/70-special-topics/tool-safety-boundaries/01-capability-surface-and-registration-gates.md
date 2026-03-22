# 工具能力面与注册闸门

## 起点与终点

- 起点：`AgentLoop._register_default_tools` 组装工具集
- 终点：模型只看到“被注册且通过参数校验”的能力集合

## 安全总分层

NanoBot 的工具安全不是单点防护，而是四层叠加：

1. 注册层：哪些工具会被注册
2. 配置层：哪些能力可被关闭或收敛
3. 参数层：调用参数是否符合 schema
4. 执行层：每个工具内部的边界检查

源码锚点：

- 注册入口：[loop.py:L116-L135](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L116-L135)
- 参数校验：[registry.py:L38-L59](file:///Users/bowhead/nanobot/nanobot/agent/tools/registry.py#L38-L59)
- schema 校验实现：[base.py:L138-L188](file:///Users/bowhead/nanobot/nanobot/agent/tools/base.py#L138-L188)

## 默认工具面与闸门

- 文件工具始终注册，但可被 `restrict_to_workspace` 全局收敛
- `exec` 只有在 `tools.exec.enable=true` 时才注册
- `cron` 只有 gateway 注入了 `CronService` 才注册
- `web_search/web_fetch/message/spawn` 默认可用，风险控制主要在工具内部

源码锚点：

- `restrict_to_workspace` 注入：[loop.py:L118-L129](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L118-L129)
- `exec` 注册开关：[loop.py:L123-L130](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L123-L130)
- `cron` 注册条件：[loop.py:L134-L135](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L134-L135)
- 配置模型：[schema.py:L118-L143](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L118-L143)

## 外部 MCP 的最小暴露面

MCP server 不会把全部能力自动裸露给模型，`enabled_tools` 是关键阀门：

- `["*"]`：放行全部 MCP 工具
- `[]`：不注册任何 MCP 工具
- 指定列表：仅暴露白名单工具

源码锚点：

- MCP 工具过滤：[mcp.py:L141-L163](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L141-L163)
- 未匹配白名单告警：[mcp.py:L170-L180](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L170-L180)
- MCP 配置字段：[schema.py:L125-L136](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L125-L136)

## 结论

- 真正高风险面主要在 `exec`、文件写操作、副作用工具和外部 MCP
- NanoBot 已有“可关闭 + 可收敛 + 可校验”的基础安全骨架
- 生产环境应把 `restrict_to_workspace=true` 作为默认策略
