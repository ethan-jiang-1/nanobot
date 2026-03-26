# workspace MD 契约：每个文件该做什么（Do / Do Not / 样板）

## 起点与终点

- 起点：开发者希望把 workspace 里的几个 MD 用对，不让 Agent 行为漂移
- 终点：按文件拿到“职责、Do、Do Not、模板来源、代码约束强度”清单

## 全局结论

- `AGENTS.md/SOUL.md/USER.md/TOOLS.md` 是 bootstrap 指令层，注入 system prompt，属于文本契约
- `HEARTBEAT.md` 是运行时心跳任务层，不进 bootstrap 注入，由 HeartbeatService 定时读取
- `memory/MEMORY.md` 与 `memory/HISTORY.md` 是持久记忆层，由 MemoryStore 直接读写

源码锚点：

- bootstrap 清单与加载：[context.py:L19-L20](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L19-L20), [context.py:L109-L119](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L109-L119)
- heartbeat 读取：[service.py:L73-L83](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L73-L83)
- memory 读写：[memory.py:L72-L99](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L72-L99)

## AGENTS.md

- 作用：定义“Agent 如何执行任务”的一线规则，优先放策略与流程性约束
- Do：
  - 写执行策略、任务优先级、提醒/周期任务原则
  - 写跨工具行为规则（例如先检查技能，再调用某工具）
  - 保持可执行、可验证、少歧义
- Do Not：
  - 不写用户画像（放 `USER.md`）
  - 不写人格口吻（放 `SOUL.md`）
  - 不写一次性任务记录（放会话或 `HEARTBEAT.md`/memory）
- 样板：有默认模板  
  [templates/AGENTS.md](file:///Users/bowhead/nanobot/nanobot/templates/AGENTS.md)
- 代码约束强度：弱约束（文本注入，不做 schema 校验）

## SOUL.md

- 作用：定义人格、价值观、沟通风格
- Do：
  - 写语气、价值偏好、输出风格边界
  - 写稳定且长期有效的沟通原则
- Do Not：
  - 不写具体项目流程步骤
  - 不写用户个人资料
  - 不写工具参数细则
- 样板：有默认模板  
  [templates/SOUL.md](file:///Users/bowhead/nanobot/nanobot/templates/SOUL.md)
- 代码约束强度：弱约束（文本注入）

## USER.md

- 作用：定义用户画像与偏好，影响回答颗粒度与表达方式
- Do：
  - 写语言偏好、技术水平、工作背景、关注主题
  - 写“输出长短”“专业程度”这类偏好
- Do Not：
  - 不写系统级安全边界
  - 不写工具调用底层约束
  - 不写临时会话状态
- 样板：有默认模板  
  [templates/USER.md](file:///Users/bowhead/nanobot/nanobot/templates/USER.md)
- 代码约束强度：弱约束（文本注入）

## TOOLS.md

- 作用：补充工具层“非显而易见”规则，尤其是安全与边界提醒
- Do：
  - 写工具限制、超时、风险边界、推荐用法
  - 写项目特有工具注意事项
- Do Not：
  - 不重复工具 schema 已声明的字段
  - 不写人格/用户偏好内容
  - 不把它当任务清单文件使用
- 样板：有默认模板  
  [templates/TOOLS.md](file:///Users/bowhead/nanobot/nanobot/templates/TOOLS.md)
- 代码约束强度：弱约束（文本注入）

## HEARTBEAT.md

- 作用：维护周期任务清单，供 heartbeat 周期检查与触发
- Do：
  - 写周期性、可复用的任务条目
  - 任务完成后迁移到 Completed 或清理
- Do Not：
  - 不写一次性提醒（优先用 `cron`）
  - 不把它当系统提示词文件
  - 不假设写入后必定外发消息（仍有评估与通知门控）
- 样板：有默认模板  
  [templates/HEARTBEAT.md](file:///Users/bowhead/nanobot/nanobot/templates/HEARTBEAT.md)
- 代码约束强度：中约束（文件存在与内容会影响运行时行为）

源码锚点：

- tick 时读取并判空：[service.py:L143-L150](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L143-L150)
- 决策与执行链路：[service.py:L85-L110](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L85-L110), [service.py:L154-L174](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L154-L174)

## memory/MEMORY.md

- 作用：长期记忆的当前全量版本
- Do：
  - 保留稳定事实与长期偏好
  - 让新状态覆盖旧状态，保持“当前真相”
- Do Not：
  - 不写分钟级流水日志
  - 不混入无关临时调试噪声
- 样板：模板会自动补齐  
  [templates/memory/MEMORY.md](file:///Users/bowhead/nanobot/nanobot/templates/memory/MEMORY.md)
- 代码约束强度：强约束（MemoryStore 直接读写）

## memory/HISTORY.md

- 作用：grep 友好的历史归档日志
- Do：
  - 记录关键事件/决策摘要
  - 保留时间前缀，便于检索
- Do Not：
  - 不把它当长期事实存储
  - 不手工破坏时间序列格式
- 样板：由模板同步创建空文件并持续追加  
  [helpers.py:L281-L283](file:///Users/bowhead/nanobot/nanobot/utils/helpers.py#L281-L283)
- 代码约束强度：强约束（append-only 语义）

源码锚点：

- append 历史：[memory.py:L94-L99](file:///Users/bowhead/nanobot/nanobot/agent/memory.py#L94-L99)

## 模板来源与覆盖策略

- 模板来源：`nanobot/templates`
- 同步策略：只补缺，不覆盖已有文件
- 调用时机：`onboard`、`gateway`、`agent` 三个入口都会触发

源码锚点：

- 同步实现：[helpers.py:L259-L289](file:///Users/bowhead/nanobot/nanobot/utils/helpers.py#L259-L289)
- 调用入口：[commands.py:L314-L321](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L314-L321), [commands.py:L509-L514](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L509-L514), [commands.py:L702-L704](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L702-L704)
