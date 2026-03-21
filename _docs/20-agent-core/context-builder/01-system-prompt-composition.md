# system prompt 拼装机制

## 目标

这篇拆解 `ContextBuilder.build_system_prompt` 如何把多个来源拼成稳定的 system prompt。

## 拼装顺序

`build_system_prompt` 采用固定顺序追加 `parts`，最后用 `\n\n---\n\n` 连接：

1. identity 基础身份块
2. bootstrap 文件块
3. Memory 上下文块
4. Active Skills 块
5. Skills 摘要块

源码锚点：

- `build_system_prompt`：[context.py:L27-L54](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L27-L54)

## identity 块包含什么

identity 不是一句人格描述，而是完整运行时元信息与行为约束：

- 运行平台与 Python 版本
- workspace 路径、memory 路径、skills 路径
- 平台策略（Windows 与 POSIX 分流）
- nanobot guidelines

源码锚点：

- `_get_identity`：[context.py:L56-L98](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L56-L98)

## bootstrap 文件加载策略

`BOOTSTRAP_FILES = ["AGENTS.md", "SOUL.md", "USER.md", "TOOLS.md"]`，按顺序读取 workspace 下同名文件，存在才纳入。

格式为：

- `## 文件名`
- 文件正文

源码锚点：

- 常量与读取函数：[context.py:L19-L20](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L19-L20)、[context.py:L108-L118](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L108-L118)

相关测试：

- 模板完整性：[test_context_prompt_cache.py:L27-L32](file:///Users/bowhead/nanobot/tests/test_context_prompt_cache.py#L27-L32)

## Memory 与 Skills 如何进入 prompt

- Memory：读取 `MemoryStore.get_memory_context()`，非空时加 `# Memory`
- Active Skills：读取 always skills 的完整内容
- Skills：加入技能摘要列表，指导“先 read_file 再使用技能”

这使模型可同时获得“长期事实”和“能力目录”。

## 稳定性约束

system prompt 里不包含当前时刻，避免分钟级抖动破坏缓存命中。

相关测试：

- 时钟变化前后一致：[test_context_prompt_cache.py:L34-L48](file:///Users/bowhead/nanobot/tests/test_context_prompt_cache.py#L34-L48)

## 设计结论

- system prompt 承担“稳定规则层”
- 时变信息放到 runtime user 侧注入
- 这是一种“缓存友好 + 安全分层”的提示词架构
