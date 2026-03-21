# 新能力接入清单（tools and execution 视角）

## 目标

当你要给 NanoBot 增加新能力时，优先选择“最小侵入、可观测、可回退”的接入路径。

## 路径选择

1. 先判断是否可用已有工具 + 新 skill 解决  
2. 若不能，再新增内置 Tool  
3. 若能力来自外部系统，优先走 MCP server 接入

## 接入检查项

- 是否定义了清晰的参数 schema（含 required/range/enum）
- 是否具备最小安全边界（路径、网络、权限、超时）
- 失败返回是否可被模型理解并自修复
- 是否需要会话上下文注入（如 channel/chat_id）
- 是否有测试覆盖关键分支（成功、超时、错误、非法参数）

## 代码落点参考

- Tool 抽象与参数校验：[base.py:L38-L199](file:///Users/bowhead/nanobot/nanobot/agent/tools/base.py#L38-L199)
- 注册与分发：[registry.py:L18-L59](file:///Users/bowhead/nanobot/nanobot/agent/tools/registry.py#L18-L59)
- 默认工具注册入口：[loop.py:L116-L135](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L116-L135)
- MCP 外部接入：[mcp.py:L74-L184](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L74-L184)

## 文档落点建议

- 如果是运行时协议问题，写入 `tool-registry-and-runtime`
- 如果是内置能力边界，写入 `built-in-tools-and-guards`
- 如果是外部 server 接入，写入 `mcp-external-tools`
- 如果是任务方法论封装，写入 `skills-and-extension-points`
