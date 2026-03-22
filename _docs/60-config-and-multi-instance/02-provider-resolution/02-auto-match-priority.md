# auto 匹配优先级

## 起点与终点

- 起点：`agents.defaults.provider = "auto"`
- 终点：返回 `(ProviderConfig, provider_name)` 或 `(None, None)`

## `_match_provider` 的优先级顺序

1. 显式模型前缀匹配：`provider/model` 里的前缀优先
2. 关键字匹配：按 registry 顺序扫描 `spec.keywords`
3. local provider 回退：有 `api_base` 的本地 provider 优先
4. 通用凭证回退：按 registry 顺序选择首个有 `api_key` 的非 OAuth provider

源码锚点：

- 前缀与关键字匹配：[schema.py:L171-L193](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L171-L193)
- local 回退：[schema.py:L194-L211](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L194-L211)
- 通用回退：[schema.py:L212-L220](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L212-L220)

## 为什么“前缀优先”

- 可避免 `github-copilot/...codex` 被误判为 `openai_codex`
- 模型明确声明来源时，应覆盖模糊关键字匹配

## API base 的衍生规则

`get_api_base` 在命中 provider 后：

- 若用户已填 `api_base`，直接使用
- 否则仅对 gateway/local provider 使用 registry 默认 `default_api_base`

源码锚点：

- `get_api_base`：[schema.py:L237-L251](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L237-L251)
