# Cron 存储模型与持久化

## 起点与终点

- 起点：CronService 启动或任一 API 首次访问 `_load_store`
- 终点：内存态 `CronStore` 与磁盘 `jobs.json` 达成同步

## 数据模型分层

Cron 的状态不是散落字段，而是分层数据结构：

- `CronSchedule`：定义触发方式（`at/every/cron`）
- `CronPayload`：定义任务内容与投递目标
- `CronJobState`：定义下一次触发、最近执行、历史记录
- `CronJob`：聚合 schedule/payload/state
- `CronStore`：聚合全部 jobs 与版本号

源码锚点：

- 类型定义：[types.py:L7-L69](file:///Users/bowhead/nanobot/nanobot/cron/types.py#L7-L69)

## 加载策略

`_load_store` 采用“惰性加载 + 外部变更感知”：

- 若已有内存 store，则先比对文件 mtime
- mtime 改变时丢弃缓存并重载
- 首次加载时把 JSON 映射回 dataclass 结构
- 解析失败时降级为空 `CronStore`，避免服务崩溃

源码锚点：

- 加载与外部重载：[service.py:L80-L139](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L80-L139)

## 保存策略

`_save_store` 采用“全量快照写入”：

- 先确保父目录存在
- 把内存 jobs 全量序列化为 `jobs` 数组
- 写入后回填 `_last_mtime`，供后续热重载判断

这种设计避免局部 patch 带来的一致性复杂度。

源码锚点：

- 保存实现：[service.py:L141-L193](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L141-L193)

## 实例级路径语义

Cron 文件不放 workspace，而放“实例数据目录”下的 `cron/jobs.json`：

- `get_cron_dir()` 返回实例级 cron 目录
- gateway 启动时据此创建 `CronService(store_path=.../jobs.json)`

这保证多实例互不污染。

源码锚点：

- 路径函数：[paths.py:L27-L29](file:///Users/bowhead/nanobot/nanobot/config/paths.py#L27-L29)
- gateway 注入路径：[commands.py:L520-L522](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L520-L522)
