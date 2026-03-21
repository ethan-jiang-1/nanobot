# Provider、Bus 与 AgentLoop 装配关系

## 起点与终点

- 起点：gateway 进入对象创建阶段
- 终点：`AgentLoop` 拿到完整依赖并可处理消息

## wiring 核心

在 gateway 中，`MessageBus`、Provider、`SessionManager` 会先创建，然后统一注入 `AgentLoop`：

- `bus`：承接 channel 与 agent 的解耦通信
- `provider`：承接模型调用
- `session_manager`：承接持久化会话
- 其余参数：工具配置、MCP、模型、上下文窗口

源码锚点：

- AgentLoop 装配：[commands.py:L520-L544](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L520-L544)

## AgentLoop 内部接线

`AgentLoop.__init__` 再把注入依赖挂到内部组件：

- `self.context = ContextBuilder(workspace)`
- `self.sessions = session_manager or SessionManager(workspace)`
- `self.tools = ToolRegistry()`
- `self.memory_consolidator = MemoryConsolidator(...)`

源码锚点：

- 初始化接线：[loop.py:L52-L114](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L52-L114)

## Bus 语义

`MessageBus` 只维护两个异步队列：

- inbound：channel -> agent
- outbound：agent -> channel

这让 channel SDK 生命周期与 Agent 核心推理逻辑解耦。

源码锚点：

- 队列定义：[queue.py:L8-L44](file:///Users/bowhead/nanobot/nanobot/bus/queue.py#L8-L44)

## 相关测试

- gateway 默认 workspace 取配置：[test_commands.py:L475-L503](file:///Users/bowhead/nanobot/tests/test_commands.py#L475-L503)
- `--workspace` 覆盖配置：[test_commands.py:L505-L534](file:///Users/bowhead/nanobot/tests/test_commands.py#L505-L534)

