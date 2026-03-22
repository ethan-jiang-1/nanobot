# 配置入口与活动 config path

## 起点与终点

- 起点：用户执行 `nanobot onboard/agent/gateway ... --config ... --workspace ...`
- 终点：运行时拿到 `Config` 对象，且“活动配置路径”被确定

## CLI 入口分成两条链路

- 初始化链路：`onboard(...)`
- 运行时链路：`_load_runtime_config(...)`

两者都支持 `--config`，但行为不同：

- `onboard`：关注“创建/刷新/写回配置文件”
- `_load_runtime_config`：关注“读取配置并给本次进程使用”

源码锚点：

- onboard 入口：[commands.py:L264-L352](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L264-L352)
- 运行时加载入口：[commands.py:L451-L468](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L451-L468)

## config path 的确定规则

- 显式传入 `--config`：先 `expanduser().resolve()`，然后调用 `set_config_path(...)`
- 未传入 `--config`：走 `get_config_path()` 默认 `~/.nanobot/config.json`
- 后续实例目录推导都依赖这个“活动路径”

源码锚点：

- `set_config_path/get_config_path`：[loader.py:L15-L26](file:///Users/bowhead/nanobot/nanobot/config/loader.py#L15-L26)
- `_load_runtime_config` 的 `--config` 处理：[commands.py:L455-L463](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L455-L463)

## workspace 覆盖规则

- `--workspace` 只覆盖当前 `Config` 对象的 `agents.defaults.workspace`
- 该覆盖不自动写回配置文件（除非走 onboard 并 save）
- 运行期路径解析使用 `config.workspace_path`

源码锚点：

- workspace 覆盖：[commands.py:L466-L468](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L466-L468)
- `workspace_path` 属性：[schema.py:L155-L159](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L155-L159)

## 关键边界

- `--config` 影响实例级 runtime 数据目录（cron/media/logs 等）
- `--workspace` 影响会话与记忆等工作目录
- 两者是“可分离维度”，因此同一 config 可临时指向不同 workspace
