# instance-isolation

本目录聚焦“路径即边界”：哪些状态跟随实例，哪些状态全局共享，以及历史数据如何迁移。

## 本主题要回答的问题

- 为什么 `--config` 会改变 cron/media/logs 等运行时目录
- workspace 与实例数据目录的边界如何划分
- 哪些路径是全局共享而不是实例隔离
- session 的 legacy 路径如何迁移到当前 workspace

## 建议文档拆分

- `01-instance-data-root-derivation.md`：实例根目录如何由 config path 推导
- `02-runtime-subdirs-and-channel-state.md`：cron/media/channel 状态目录的隔离语义
- `03-shared-vs-instance-scoped-paths.md`：全局路径与实例路径对照
- `04-session-storage-and-legacy-migration.md`：session 目录组织与旧路径迁移

## 分析抓手

- `get_data_dir` / `get_runtime_subdir` / `get_workspace_path`
- `get_cli_history_path` / `get_bridge_install_dir` / `get_legacy_sessions_dir`
- `SessionManager._load` legacy 迁移分支
