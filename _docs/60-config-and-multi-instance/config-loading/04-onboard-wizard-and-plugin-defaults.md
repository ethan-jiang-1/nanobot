# onboard 向导与插件默认配置注入

## 起点与终点

- 起点：执行 `nanobot onboard --wizard` 或普通 `nanobot onboard`
- 终点：配置被保存，并补齐已发现 channel 的默认字段

## wizard 的输入与输出契约

- `run_onboard(initial_config=...)` 接收初始配置副本
- 返回 `OnboardResult(config, should_save)`
- `should_save=False` 时，onboard 外层直接丢弃变更

源码锚点：

- 返回结构：[onboard_wizard.py:L31-L35](file:///Users/bowhead/nanobot/nanobot/cli/onboard_wizard.py#L31-L35)
- 主入口与退出分支：[onboard_wizard.py:L956-L1019](file:///Users/bowhead/nanobot/nanobot/cli/onboard_wizard.py#L956-L1019)
- onboard 对结果的处理：[commands.py:L309-L326](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L309-L326)

## Channel 配置来源

- wizard 中 channel 菜单来自 `discover_all()`
- 每个 channel 配置类由模块内 `*Config` 模型反射获取
- 用户修改后以 `model_dump(by_alias=True)` 回写到 `config.channels.<name>`

源码锚点：

- channel 发现：[onboard_wizard.py:L742-L759](file:///Users/bowhead/nanobot/nanobot/cli/onboard_wizard.py#L742-L759)
- 单 channel 编辑回写：[onboard_wizard.py:L773-L796](file:///Users/bowhead/nanobot/nanobot/cli/onboard_wizard.py#L773-L796)

## 插件默认配置注入机制

无论是否 wizard，`onboard` 最后都会执行 `_onboard_plugins(config_path)`：

- 读取现有 `config.json`
- 遍历 `discover_all()` 返回的 channel
- 若 channel 不存在则写入 `default_config()`
- 若已存在则递归补齐缺失字段，不覆盖用户已设值

源码锚点：

- `_onboard_plugins` 主流程：[commands.py:L368-L390](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L368-L390)
- 递归补齐策略：[commands.py:L354-L365](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L354-L365)

## 关键边界

- wizard 负责“交互编辑”
- `_onboard_plugins` 负责“结构补齐”
- 两者组合保证：新插件接入后，旧配置也能自动补出必需字段
