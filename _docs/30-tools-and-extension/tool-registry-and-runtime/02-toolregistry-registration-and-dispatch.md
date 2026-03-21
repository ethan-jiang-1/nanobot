# ToolRegistry 注册与分发

## 起点与终点

- 起点：`AgentLoop` 或其他组件向 `ToolRegistry.register()` 注册工具
- 终点：模型可见工具定义，并可通过 `execute(name, params)` 统一调用

## 注册模型

`ToolRegistry` 内部是 `dict[str, Tool]`：

- 注册同名工具会覆盖旧值
- `unregister/get/has` 提供基础管理能力

源码锚点：

- 注册与管理：[registry.py:L15-L33](file:///Users/bowhead/nanobot/nanobot/agent/tools/registry.py#L15-L33)

## 暴露给模型的定义

`get_definitions()` 将当前注册工具映射成 schema 列表，直接用于 provider 的 `tools` 参数。

源码锚点：

- 定义导出：[registry.py:L34-L36](file:///Users/bowhead/nanobot/nanobot/agent/tools/registry.py#L34-L36)

## 执行分发与错误语义

`execute(name, params)` 包含完整分发链：

1. 按名称取工具，不存在时返回可用工具列表
2. 参数 cast + validate
3. 调用具体工具 `execute`
4. 对错误结果追加提示，鼓励模型采用替代方案

这使 registry 不只做路由，还提供“面向模型的错误反馈协议”。

源码锚点：

- 执行分发：[registry.py:L38-L59](file:///Users/bowhead/nanobot/nanobot/agent/tools/registry.py#L38-L59)

## 设计含义

- 优点：模型看到一致错误格式，更容易自修复
- 边界：registry 本身不做重试/回滚，只负责分发与错误包装
