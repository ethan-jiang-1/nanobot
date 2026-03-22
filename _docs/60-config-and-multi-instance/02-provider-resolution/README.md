# provider-resolution

本目录聚焦 Provider 路由决策：配置如何匹配到 provider、如何落到具体运行时实现。

## 本主题要回答的问题

- `agents.defaults.provider` 为 `auto` 与非 `auto` 时行为差异
- model 字符串如何触发 provider 关键字匹配与前缀匹配
- gateway/local provider 如何通过 api_key/api_base 参与兜底
- 最终何时走 LiteLLM，何时走 direct provider

## 建议文档拆分

- `01-provider-schema-and-forced-mode.md`：schema 层 provider 字段与强制模式
- `02-auto-match-priority.md`：`_match_provider` 自动匹配优先级
- `03-gateway-local-fallback.md`：registry 的 gateway/local 识别与回退
- `04-runtime-provider-instantiation.md`：`_make_provider` 与 LiteLLM 模型改写

## 分析抓手

- `Config._match_provider/get_provider_name/get_api_base`
- `PROVIDERS/find_by_model/find_gateway/find_by_name`
- `_make_provider` / `LiteLLMProvider._resolve_model`
