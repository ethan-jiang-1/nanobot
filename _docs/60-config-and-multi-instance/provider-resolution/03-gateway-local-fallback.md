# gateway/local 识别与回退

## 起点与终点

- 起点：系统需要判断是否启用 gateway 或 local provider 路由
- 终点：得到 gateway/local 的 `ProviderSpec`，或返回 `None`

## registry 是单一事实源

- `PROVIDERS` 声明 provider 元信息与优先顺序
- 顺序即优先级，gateway 放前面用于兜底路由
- `ProviderSpec` 同时定义关键字、env 映射、前缀策略与默认 base

源码锚点：

- registry 与 ProviderSpec：[registry.py:L19-L73](file:///Users/bowhead/nanobot/nanobot/providers/registry.py#L19-L73)
- PROVIDERS 定义：[registry.py:L73-L457](file:///Users/bowhead/nanobot/nanobot/providers/registry.py#L73-L457)

## find_gateway 的识别顺序

1. `provider_name` 显式命中 gateway/local
2. `api_key` 前缀命中（例如 `sk-or-`）
3. `api_base` 关键字命中（例如 `aihubmix`、`11434`）

源码锚点：

- `find_gateway`：[registry.py:L487-L515](file:///Users/bowhead/nanobot/nanobot/providers/registry.py#L487-L515)

## find_by_model 的边界

- `find_by_model` 只匹配标准 provider（排除 gateway/local）
- 先看显式前缀，再看关键字

源码锚点：

- `find_by_model`：[registry.py:L465-L484](file:///Users/bowhead/nanobot/nanobot/providers/registry.py#L465-L484)

## 设计收益

- 网关识别不再依赖分散 if/elif
- “新增 provider”只需改 registry + schema 两处
