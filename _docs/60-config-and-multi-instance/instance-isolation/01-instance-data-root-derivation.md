# 实例数据根目录推导

## 起点与终点

- 起点：进程已确定活动 config path
- 终点：实例级 runtime 根目录被解析为 `config_path.parent`

## 推导规则

实例数据目录不依赖 workspace，而依赖配置文件所在目录：

- `get_config_path()` 给出活动配置文件
- `get_data_dir()` 直接返回其父目录并确保存在

这意味着：切换 `--config` 就会切换实例 runtime 目录。

源码锚点：

- 配置路径来源：[loader.py:L21-L26](file:///Users/bowhead/nanobot/nanobot/config/loader.py#L21-L26)
- 实例根目录推导：[paths.py:L11-L14](file:///Users/bowhead/nanobot/nanobot/config/paths.py#L11-L14)

## 与 workspace 的关系

- `workspace` 是 agent 工作目录，来自 config 或 `--workspace`
- `data_dir` 是实例运行状态目录，来自 config 文件位置
- 两者可同向也可分离，分别承载不同状态

源码锚点：

- workspace 解析：[paths.py:L37-L40](file:///Users/bowhead/nanobot/nanobot/config/paths.py#L37-L40)
- workspace 覆盖：[commands.py:L466-L468](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L466-L468)

## 校验基线

测试明确断言了该规则：

- `get_data_dir() == config_file.parent`
- `get_cron_dir/get_logs_dir/get_runtime_subdir` 均挂在该目录下

源码锚点：

- 测试用例：[test_config_paths.py:L16-L24](file:///Users/bowhead/nanobot/tests/test_config_paths.py#L16-L24)
