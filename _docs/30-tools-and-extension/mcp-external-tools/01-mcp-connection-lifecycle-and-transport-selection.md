# MCP 连接生命周期与传输模式选择

## 起点与终点

- 起点：配置中存在 `tools.mcp_servers`
- 终点：每个可连接 server 完成初始化并暴露可注册工具列表

## 连接入口

MCP 连接在 `connect_mcp_servers(mcp_servers, registry, stack)` 内统一处理，按 server 顺序逐个尝试，单个失败不影响其他 server。

源码锚点：

- 连接入口：[mcp.py:L74-L84](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L74-L84)
- 单 server 异常隔离：[mcp.py:L183-L184](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L183-L184)

## 传输模式选择规则

若未显式设置 `type`，自动推断：

- 有 `command`：`stdio`
- 有 `url` 且后缀 `/sse`：`sse`
- 其余 `url`：`streamableHttp`

源码锚点：

- 自动推断逻辑：[mcp.py:L85-L97](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L85-L97)
- 三种连接分支：[mcp.py:L98-L133](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L98-L133)

## 会话初始化

传输建立后，流程固定为：

1. `ClientSession(read, write)`
2. `session.initialize()`
3. `session.list_tools()`

源码锚点：

- 初始化与列工具：[mcp.py:L137-L141](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L137-L141)
