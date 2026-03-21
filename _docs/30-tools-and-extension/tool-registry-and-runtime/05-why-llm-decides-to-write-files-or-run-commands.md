# 为什么模型会决定“写文件”或“执行命令”

## 决策输入来自哪里

模型每一轮拿到三类关键信号：

1. 当前对话目标（用户要“解释”还是“交付产物”）
2. system prompt 与 bootstrap 约束（workspace、行为规则、项目偏好）
3. 本轮可用工具 schema（名称、描述、参数）

源码锚点：

- tools 传入 provider：[loop.py:L198-L204](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L198-L204)
- tool schema 生成：[base.py:L190-L199](file:///Users/bowhead/nanobot/nanobot/agent/tools/base.py#L190-L199)
- registry 导出定义：[registry.py:L34-L35](file:///Users/bowhead/nanobot/nanobot/agent/tools/registry.py#L34-L35)
- provider 采用 `tool_choice=auto`：[openai_codex_provider.py:L44-L66](file:///Users/bowhead/nanobot/nanobot/providers/openai_codex_provider.py#L44-L66)

## “写文件”倾向如何形成

当用户目标是“生成代码/配置/文档产物”时，`write_file` 与 `edit_file` 的描述和参数天然匹配该目标，模型通常会优先调用它们。

源码锚点：

- `write_file` 契约：[filesystem.py:L142-L173](file:///Users/bowhead/nanobot/nanobot/agent/tools/filesystem.py#L142-L173)
- `edit_file` 契约：[filesystem.py:L206-L240](file:///Users/bowhead/nanobot/nanobot/agent/tools/filesystem.py#L206-L240)

## “执行命令”倾向如何形成

当目标包含“验证是否可运行/可通过测试/可安装依赖”时，模型通常会继续调用 `exec` 来获取真实执行反馈。

源码锚点：

- `exec` 参数语义：[shell.py:L52-L76](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L52-L76)
- `exec` 实际执行与输出回传：[shell.py:L78-L142](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L78-L142)

## 错误反馈如何推动下一步决策

ToolRegistry 在参数错误、执行异常、工具返回 Error 时都会把错误文本回传给模型，并追加“分析错误后换方案”的提示。  
这让模型更容易进入“修复-重试”路径，而不是停在失败点。

源码锚点：

- execute 错误处理与提示注入：[registry.py:L37-L59](file:///Users/bowhead/nanobot/nanobot/agent/tools/registry.py#L37-L59)

## 边界条件会反过来塑造决策

如果 `exec` 命令被安全策略拦截，或者路径超出工作区，模型会收到错误并转向更保守方案（如仅修改文件、调整命令、缩小作用域）。

源码锚点：

- `exec` 安全拦截：[shell.py:L144-L176](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L144-L176)
- 文件路径约束：[filesystem.py:L10-L25](file:///Users/bowhead/nanobot/nanobot/agent/tools/filesystem.py#L10-L25)
