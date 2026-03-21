# skills 发现优先级与可用性过滤

## 起点与终点

- 起点：`SkillsLoader.list_skills()` 被调用
- 终点：返回可用于当前运行环境的技能列表

## 发现顺序

skills 发现顺序是：

1. workspace `skills/{name}/SKILL.md`
2. 内置 `nanobot/skills/{name}/SKILL.md`（若同名则被 workspace 覆盖）

源码锚点：

- 发现与同名覆盖：[skills.py:L26-L57](file:///Users/bowhead/nanobot/nanobot/agent/skills.py#L26-L57)

## 可用性过滤

`filter_unavailable=True` 时，会用 `_check_requirements` 过滤掉环境不满足的技能。  
检查维度包括：

- 必需可执行命令是否在 PATH
- 必需环境变量是否存在

源码锚点：

- 可用性检查：[skills.py:L177-L186](file:///Users/bowhead/nanobot/nanobot/agent/skills.py#L177-L186)
- 缺失项描述：[skills.py:L142-L152](file:///Users/bowhead/nanobot/nanobot/agent/skills.py#L142-L152)

## 设计含义

- workspace 可覆盖内置，便于团队在本地定制行为
- 可用性过滤让模型尽量只看到可落地技能，减少失败尝试
