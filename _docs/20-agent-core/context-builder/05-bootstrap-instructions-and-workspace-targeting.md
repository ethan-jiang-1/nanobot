# bootstrap 指令与产物路径定位来源

## 起点与终点

- 起点：`build_system_prompt()` 构造 system prompt
- 终点：模型在同一轮里同时拿到 workspace 路径、bootstrap 规则、skills 摘要

## 路径感知是如何注入的

ContextBuilder 会把 workspace 绝对路径直接写进 `## Workspace` 段落。  
这让模型在没有额外猜测的情况下，就能围绕该目录组织“读/写/执行”动作。

源码锚点：

- identity 与 workspace 注入：[context.py:L56-L87](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L56-L87)

## bootstrap 文件如何影响“写到哪里”

`AGENTS.md`、`SOUL.md`、`USER.md`、`TOOLS.md` 若存在于 workspace，会被原样拼入系统提示词。  
因此项目可以通过这些文件声明偏好的输出目录、交付流程或执行策略。

源码锚点：

- bootstrap 文件清单：[context.py:L19-L20](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L19-L20)
- 文件加载并拼接：[context.py:L108-L118](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L108-L118)

## 这些文件从哪里来

初始化阶段会把 `nanobot/templates` 下的模板同步到 workspace（仅补缺，不覆盖）。  
这保证新 workspace 默认就具备可被注入的行为约束文件。

源码锚点：

- 初始化时同步模板：[commands.py:L338-L351](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L338-L351)
- 同步实现（仅创建缺失文件）：[helpers.py:L179-L211](file:///Users/bowhead/nanobot/nanobot/utils/helpers.py#L179-L211)

## 为什么不是“固定目录写死”

系统没有写“必须输出到某个固定子目录”的硬编码。  
实际效果是：

- workspace 提供默认边界
- bootstrap 提供项目约束
- tool schema 提供可行动作

三者合并后，模型才形成“该写哪、该不该执行”的策略。
