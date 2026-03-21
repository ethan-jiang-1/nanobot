# mcp-external-tools

本目录聚焦外部 MCP 工具如何被接入到 NanoBot：连接、注册、过滤、生命周期管理。

## 本主题要回答的问题

- MCP server 如何自动判断传输模式并连接
- 外部工具如何包装成统一 Tool 接口
- enabledTools 如何按 raw/wrapped 名称过滤
- MCP 连接失败、工具超时、shutdown 时如何处理

## 建议文档拆分

- `01-mcp-connection-lifecycle-and-transport-selection.md`
- `02-mcp-tool-wrapper-timeout-cancel-error.md`
- `03-enabled-tools-filtering-and-name-mapping.md`
- `04-agentloop-mcp-lazy-connect-and-shutdown.md`
