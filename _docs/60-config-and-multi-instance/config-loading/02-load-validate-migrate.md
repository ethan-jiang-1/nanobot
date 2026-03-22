# 加载、校验与迁移

## 起点与终点

- 起点：`load_config(config_path)` 被调用
- 终点：返回合法 `Config`；若失败则降级默认配置

## load_config 的主流程

- 先选路径：优先参数 `config_path`，否则 `get_config_path()`
- 文件存在时读取 JSON，并先过 `_migrate_config`
- 然后 `Config.model_validate(data)` 进入 schema 校验
- 解析失败（JSON/值错误/校验错误）则告警并回退到 `Config()`

源码锚点：

- 主流程：[loader.py:L28-L50](file:///Users/bowhead/nanobot/nanobot/config/loader.py#L28-L50)

## schema 校验的关键点

- 基础模型支持 camelCase 与 snake_case 混用
- Root `Config` 聚合 agents/channels/providers/gateway/tools
- `BaseSettings` 启用 `NANOBOT_` 前缀与嵌套环境变量分隔符

源码锚点：

- `Base.model_config`：[schema.py:L11-L15](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L11-L15)
- `Config` 根模型：[schema.py:L146-L154](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L146-L154)
- 环境变量配置：[schema.py:L253-L253](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L253-L253)

## 兼容迁移策略

当前迁移逻辑处理旧键：

- `tools.exec.restrictToWorkspace` → `tools.restrictToWorkspace`
- 仅在新位置缺失时执行搬迁

这保证旧配置不崩溃，同时把语义收敛到新结构。

源码锚点：

- 迁移函数：[loader.py:L70-L77](file:///Users/bowhead/nanobot/nanobot/config/loader.py#L70-L77)

## 失败语义

- 读失败不会中断进程，系统以默认配置继续运行
- 这对“首次启动”和“配置损坏自愈”很有价值
- 代价是用户可能在未察觉情况下跑在默认值上，需要结合日志排查
