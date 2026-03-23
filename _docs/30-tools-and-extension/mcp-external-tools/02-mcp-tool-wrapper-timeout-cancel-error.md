# MCPToolWrapper 的超时、取消与错误语义

## 起点与终点

- 起点：某个 MCP 工具被包装为 `MCPToolWrapper`
- 终点：包装工具执行后，返回统一字符串结果或可解释错误

## 包装后的统一形态

每个 MCP 工具都会转成本地 `Tool`：

- 名称：`mcp_<server>_<tool>`
- 参数：在 `inputSchema` 基础上做 OpenAI 兼容归一化
- 描述：优先用远端 description

源码锚点：

- wrapper 初始化：[mcp.py:L17-L23](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L17-L23)
- Tool 接口实现：[mcp.py:L25-L35](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L25-L35)

## 参数 schema 归一化

对 nullable 形态会做最小改写：

- `type: ["string", "null"]` -> `type: "string", nullable: true`
- 非 dict schema 回退为最小 object schema

这样可以避免部分 provider 对 `type` 数组报错，同时保持语义不变。

## 执行时的三类异常路径

1. `TimeoutError`：返回超时提示字符串
2. `CancelledError`：区分“外部取消”与“SDK 内部取消”
3. 其他异常：记录日志并返回失败类型

这让上层模型能够在文本层面理解失败并重试不同策略。

源码锚点：

- 执行与异常处理：[mcp.py:L37-L64](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L37-L64)

## 输出规范化

MCP 返回的 content block 会被拼接为纯文本；无内容时返回 `(no output)`。

源码锚点：

- 输出拼接：[mcp.py:L65-L71](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L65-L71)

## 相关测试

- timeout/cancel/error 覆盖：[test_mcp_tool.py:L87-L154](file:///Users/bowhead/nanobot/tests/test_mcp_tool.py#L87-L154)
