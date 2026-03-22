# config-loading

本目录聚焦配置生命周期：CLI 参数进入、配置路径确定、load/save、迁移与 onboard 注入。

## 本主题要回答的问题

- `--config` 和 `--workspace` 如何影响最终运行时配置
- 配置文件损坏或字段过时时，系统如何回退与兼容
- onboard 如何刷新配置并补全插件 channel 默认字段
- 哪些字段在运行时可覆盖，哪些必须落盘

## 建议文档拆分

- `01-entrypoints-and-config-path.md`：入口命令如何确定活动 config path
- `02-load-validate-migrate.md`：load + pydantic 校验 + 兼容迁移
- `03-save-refresh-and-deprecated-keys.md`：save、刷新策略、废弃键提示
- `04-onboard-wizard-and-plugin-defaults.md`：wizard 流程与 channel 默认配置注入

## 分析抓手

- `onboard` / `_load_runtime_config` / `_warn_deprecated_config_keys`
- `set_config_path` / `get_config_path` / `load_config` / `save_config`
- `_migrate_config` / `_onboard_plugins` / `_merge_missing_defaults`
