# skill 元数据、依赖声明与 always 加载

## 起点与终点

- 起点：系统读取某个 `SKILL.md` 的 frontmatter
- 终点：得到可用于调度的 metadata，并决定是否进入 always skills

## 元数据解析

`SkillsLoader` 会读取 frontmatter 的 key-value，再从 `metadata` 字段解析 JSON，兼容：

- `nanobot`
- `openclaw`

源码锚点：

- frontmatter 读取：[skills.py:L203-L228](file:///Users/bowhead/nanobot/nanobot/agent/skills.py#L203-L228)
- metadata JSON 解析：[skills.py:L169-L175](file:///Users/bowhead/nanobot/nanobot/agent/skills.py#L169-L175)
- 统一 skill_meta 入口：[skills.py:L188-L191](file:///Users/bowhead/nanobot/nanobot/agent/skills.py#L188-L191)

## 依赖声明

skill 可通过 `requires.bins` 与 `requires.env` 声明依赖。  
这些依赖会被可用性检查和摘要展示共同使用。

源码锚点：

- 依赖检查：[skills.py:L177-L186](file:///Users/bowhead/nanobot/nanobot/agent/skills.py#L177-L186)
- 缺失依赖生成：[skills.py:L142-L152](file:///Users/bowhead/nanobot/nanobot/agent/skills.py#L142-L152)

## always 技能

`get_always_skills()` 会筛选“标记为 always 且依赖满足”的技能，作为默认激活技能注入系统提示。

源码锚点：

- always 技能筛选：[skills.py:L193-L201](file:///Users/bowhead/nanobot/nanobot/agent/skills.py#L193-L201)
