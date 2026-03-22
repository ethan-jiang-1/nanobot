# 运行时 Provider 实例化

## 起点与终点

- 起点：gateway/agent 启动，调用 `_make_provider(config)`
- 终点：返回可执行的 provider 实例，并注入 generation 默认参数

## `_make_provider` 分流

- `provider_name == openai_codex`：走 `OpenAICodexProvider`
- `provider_name == custom`：走 `CustomProvider`（绕过 LiteLLM）
- `provider_name == azure_openai`：走 `AzureOpenAIProvider`（要求 key+base）
- 其他：走 `LiteLLMProvider`

源码锚点：

- 分流逻辑：[commands.py:L392-L441](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L392-L441)

## LiteLLMProvider 的 gateway 行为

- 初始化先 `find_gateway(provider_name, api_key, api_base)`
- `_resolve_model` 中按 gateway 规则做前缀改写/剥离
- `_setup_env` 中 gateway/local 会覆盖对应 env_key

源码锚点：

- gateway 检测：[litellm_provider.py:L48-L52](file:///Users/bowhead/nanobot/nanobot/providers/litellm_provider.py#L48-L52)
- model 改写：[litellm_provider.py:L91-L109](file:///Users/bowhead/nanobot/nanobot/providers/litellm_provider.py#L91-L109)
- env 注入：[litellm_provider.py:L67-L90](file:///Users/bowhead/nanobot/nanobot/providers/litellm_provider.py#L67-L90)

## generation 参数收敛

无论选择哪类 provider，最终都统一应用：

- `temperature`
- `max_tokens`
- `reasoning_effort`

源码锚点：

- generation 设置：[commands.py:L442-L447](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L442-L447)

## 排障抓手

- 路由错配先看 `config.get_provider_name(model)` 决策结果
- 再看 LiteLLM 的 `_gateway` 是否符合预期
