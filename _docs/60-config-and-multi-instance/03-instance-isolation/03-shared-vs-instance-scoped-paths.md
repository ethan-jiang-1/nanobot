# 全局共享路径 vs 实例隔离路径

## 起点与终点

- 起点：组件需要选择存储位置
- 终点：明确“该放实例目录还是全局目录”

## 实例隔离路径

实例隔离路径跟随 `config_path.parent`：

- `data_dir`
- `cron/`
- `media/`
- `logs/`
- `runtime_subdir(<name>)`

源码锚点：

- 实例路径入口：[paths.py:L11-L34](file:///Users/bowhead/nanobot/nanobot/config/paths.py#L11-L34)

## 全局共享路径

这些路径固定在 `~/.nanobot/`，不随实例切换：

- CLI 历史：`~/.nanobot/history/cli_history`
- WhatsApp bridge 安装目录：`~/.nanobot/bridge`
- legacy sessions：`~/.nanobot/sessions`

源码锚点：

- 共享路径函数：[paths.py:L43-L55](file:///Users/bowhead/nanobot/nanobot/config/paths.py#L43-L55)

## 设计含义

- 历史命令和 bridge 安装属于“用户级资产”，共享更合理
- cron/media/logs 属于“实例级运行态”，必须隔离
- legacy sessions 保持全局是为了老版本兼容迁移

## 测试基线

- 实例路径与配置路径绑定：[test_config_paths.py:L16-L32](file:///Users/bowhead/nanobot/tests/test_config_paths.py#L16-L32)
- 全局路径保持固定：[test_config_paths.py:L34-L37](file:///Users/bowhead/nanobot/tests/test_config_paths.py#L34-L37)
