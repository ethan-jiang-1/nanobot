# runtime 子目录与 channel 状态

## 起点与终点

- 起点：组件需要持久化实例状态
- 终点：状态落盘到实例级子目录，避免跨实例串写

## 统一子目录入口

- `get_runtime_subdir(name)` 在 `get_data_dir()` 下创建并返回子目录
- `get_cron_dir/get_media_dir/get_logs_dir` 都是它的特化入口

源码锚点：

- 通用子目录函数：[paths.py:L16-L19](file:///Users/bowhead/nanobot/nanobot/config/paths.py#L16-L19)
- cron/media/logs：[paths.py:L21-L34](file:///Users/bowhead/nanobot/nanobot/config/paths.py#L21-L34)

## 典型落盘点

- Cron：`get_cron_dir() / "jobs.json"` 作为任务存储文件
- WhatsApp Bridge 登录态：`get_runtime_subdir("whatsapp-auth")`
- Mochat 游标：`get_runtime_subdir("mochat") / session_cursors.json`
- Matrix store：`get_data_dir() / "matrix-store"`

源码锚点：

- Cron 注入：[commands.py:L525-L525](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L525-L525)
- WhatsApp auth 目录：[commands.py:L987-L987](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L987-L987)
- Mochat 状态目录：[mochat.py:L278-L280](file:///Users/bowhead/nanobot/nanobot/channels/mochat.py#L278-L280)
- Matrix store 目录：[matrix.py:L201-L203](file:///Users/bowhead/nanobot/nanobot/channels/matrix.py#L201-L203)

## 关键边界

- channel/state 目录应使用 `paths.py` 统一入口
- 不应手写 `~/.nanobot/...` 固定路径，否则破坏多实例隔离
