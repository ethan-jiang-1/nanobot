# Tool 抽象与 Schema 契约

## 起点与终点

- 起点：新增一个工具类，继承 `Tool`
- 终点：该工具能被统一转成函数调用 schema，并通过统一参数流程执行

## 最小实现面

`Tool` 抽象要求每个工具提供 4 个核心能力：

1. `name`：工具名称
2. `description`：工具描述
3. `parameters`：JSON Schema 参数定义
4. `execute(**kwargs)`：异步执行入口

源码锚点：

- 抽象接口：[base.py:L38-L67](file:///Users/bowhead/nanobot/nanobot/agent/tools/base.py#L38-L67)

## 参数进入执行前的统一处理

工具参数在真正执行前会经过两层统一处理：

- `cast_params`：按 schema 尝试安全类型转换
- `validate_params`：按 schema 校验 required/type/enum/range

这让具体工具实现可以把主要精力放在业务逻辑，而不是重复做参数清洗。

源码锚点：

- cast 流程：[base.py:L69-L137](file:///Users/bowhead/nanobot/nanobot/agent/tools/base.py#L69-L137)
- validate 流程：[base.py:L138-L188](file:///Users/bowhead/nanobot/nanobot/agent/tools/base.py#L138-L188)

## 与模型函数调用的对接

所有工具都通过 `to_schema()` 变成统一格式：

- `type=function`
- `function.name / description / parameters`

因此 AgentLoop 不需要感知工具内部实现差异。

源码锚点：

- schema 转换：[base.py:L190-L199](file:///Users/bowhead/nanobot/nanobot/agent/tools/base.py#L190-L199)
