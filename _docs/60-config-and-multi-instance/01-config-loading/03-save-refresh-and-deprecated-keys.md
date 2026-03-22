# 保存、刷新与废弃键提示

## 起点与终点

- 起点：配置对象需要写回磁盘，或已有配置需要“保留值刷新”
- 终点：配置文件包含最新 schema 字段，并给出可移除废弃键提示

## save_config 的语义

- 目标路径优先取参数，否则取活动 `get_config_path()`
- 先创建父目录，再把 `model_dump(mode="json", by_alias=True)` 写成 JSON
- 输出使用 alias（camelCase），保持外部配置风格一致

源码锚点：

- save 实现：[loader.py:L53-L68](file:///Users/bowhead/nanobot/nanobot/config/loader.py#L53-L68)

## onboard 的“覆盖”与“刷新”

配置已存在且非 wizard 时，onboard 提供两种路径：

- 覆盖：`Config()` 全新默认值直接保存
- 刷新：先 `load_config` 再 `save_config`，保留已有值并回填新增字段

刷新模式的核心收益是“升级 schema 时自动补齐新字段”。

源码锚点：

- 覆盖/刷新分支：[commands.py:L286-L301](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L286-L301)

## 废弃键提示

运行时加载后会检查原始 JSON：

- 若发现 `agents.defaults.memoryWindow`
- 输出提示该键已不再使用，可安全删除

这是一种“软迁移”策略：不阻断运行，但引导清理历史配置噪音。

源码锚点：

- 提示逻辑：[commands.py:L471-L485](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L471-L485)

## 相关测试抓手

- 配置刷新补字段与 legacy 键兼容测试：[test_config_migration.py:L6-L127](file:///Users/bowhead/nanobot/tests/test_config_migration.py#L6-L127)
