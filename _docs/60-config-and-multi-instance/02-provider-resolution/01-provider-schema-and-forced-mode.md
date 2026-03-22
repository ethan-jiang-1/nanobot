# Provider schema 与强制模式

## 起点与终点

- 起点：用户在 `config.providers.*` 与 `agents.defaults.provider` 中填写配置
- 终点：系统确定“是否强制使用某 provider”或进入自动匹配

## schema 结构

- `ProvidersConfig` 声明所有 provider 配置入口
- 每个 provider 使用统一 `ProviderConfig(api_key, api_base, extra_headers)`
- `agents.defaults.provider` 默认是 `"auto"`

源码锚点：

- ProviderConfig 与 ProvidersConfig：[schema.py:L50-L83](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L50-L83)
- 默认 provider 字段：[schema.py:L34-L36](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L34-L36)

## 强制模式逻辑

`_match_provider` 最先检查 `forced = agents.defaults.provider`：

- 非 `auto`：直接按字段名取 `self.providers.<forced>`
- 找到则立即返回，不再进行模型关键字匹配
- 找不到则返回 `(None, None)`

源码锚点：

- 强制分支：[schema.py:L166-L170](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L166-L170)

## 配置约束含义

- 强制模式适合“固定供应商策略”
- auto 模式适合“根据模型名与凭证动态路由”
- 两者共用同一份 provider 凭证结构，降低配置复杂度
