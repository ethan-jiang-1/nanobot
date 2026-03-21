# AgentLoop 生命周期与装配

## 目标

这篇聚焦 AgentLoop 是如何被构造出来、启动、以及在关闭阶段做资源回收的。

## 构造阶段做了什么

初始化入口在 `AgentLoop.__init__`，这里不是简单赋值，而是完成了核心依赖装配：

- 绑定消息总线 `MessageBus`
- 绑定模型提供者 `LLMProvider`
- 创建上下文构建器 `ContextBuilder`
- 创建会话管理器 `SessionManager`
- 创建工具注册表 `ToolRegistry`
- 创建子代理管理器 `SubagentManager`
- 创建记忆收敛器 `MemoryConsolidator`
- 注册默认工具 `_register_default_tools`

对应源码锚点：

- `__init__`：[loop.py:L51-L115](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L51-L115)
- 默认工具注册：[loop.py:L116-L136](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L116-L136)

## 默认工具集合的设计意图

默认工具不是随意拼接，而是围绕“可执行代理”最小闭环：

- 文件读写与目录遍历：支持代码/文档编辑
- shell 执行（可配置开关）：支持命令验证
- web 搜索与抓取：支持外部知识检索
- message 工具：允许主动向目标会话发消息
- spawn 工具：允许拆分后台子任务
- cron 工具：在定时服务启用时可用

此外，若开启 `restrict_to_workspace`，工具会被绑定在 workspace 安全边界内。

## 启动阶段

`run` 方法是长期运行入口，启动时会先尝试 MCP 连接，再进入消息消费循环：

1. `self._running = True`
2. `await self._connect_mcp()`
3. 循环消费 inbound message
4. 分发到命令路径或正常处理路径

源码锚点：

- 运行入口：[loop.py:L257-L287](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L257-L287)
- MCP 连接：[loop.py:L137-L157](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L137-L157)

## 停止与关闭

生命周期的“结束”分两层：

- `stop`：只改变运行标志，停止继续消费新消息
- `close_mcp`：等待后台任务清空，再释放 MCP 资源

这两个步骤组合起来，保证不会“硬切”导致归档任务丢失或 MCP 状态泄漏。

源码锚点：

- `stop`：[loop.py:L358-L361](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L358-L361)
- `close_mcp`：[loop.py:L340-L350](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L340-L350)

## 模块边界

- AgentLoop 不直接操作具体 channel SDK，只通过 bus 交互
- AgentLoop 不直接实现工具细节，只依赖 `ToolRegistry.execute`
- AgentLoop 不直接做记忆摘要算法，只调用 `MemoryConsolidator`

这让 AgentLoop 成为“编排层”，而不是“实现层”。
