# 上下文注入的 skills 摘要与按需加载

## 起点与终点

- 起点：`ContextBuilder.build_system_prompt()` 构建 system prompt
- 终点：模型拿到 skills 摘要，并在需要时用 `read_file` 再加载完整技能内容

## 两段式注入策略

system prompt 对 skills 分两层注入：

1. always skills：直接注入完整内容（`# Active Skills`）
2. 全量技能摘要：注入 XML 列表，提示“需要时再 read_file”

源码锚点：

- build_system_prompt 中的 skills 逻辑：[context.py:L27-L54](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L27-L54)
- skills 摘要生成：[skills.py:L101-L140](file:///Users/bowhead/nanobot/nanobot/agent/skills.py#L101-L140)

## 渐进加载优势

- 减少初始 prompt 体积，避免把所有 SKILL.md 全量塞入上下文
- 保留 discoverability：模型仍知道有哪些技能可用
- 遇到复杂任务再按路径精准读取完整技能

## 与工具边界的耦合点

该设计依赖 `read_file` 可访问内置 skills 路径。  
在 workspace 沙箱模式下，`ReadFileTool` 通过 `extra_allowed_dirs` 允许只读访问内置技能目录。

源码锚点：

- read_file 额外白名单注入：[loop.py:L118-L121](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L118-L121)
