# 缓存、加载保存与旧路径迁移

## 目标

这篇拆解 SessionManager 的 I/O 生命周期：cache 命中、磁盘加载、保存覆盖、legacy 迁移。

## cache 命中优先

`get_or_create` 首先检查 `_cache`，命中则直接返回，避免重复磁盘读取。

源码锚点：

- [manager.py:L125-L143](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L125-L143)

## 加载流程

`_load` 读取 JSONL 时：

- 优先当前 workspace sessions 路径
- 若不存在，检查 legacy 全局路径
- 有 legacy 文件则尝试 `shutil.move` 迁移
- 再逐行解析 metadata 与 message

源码锚点：

- 迁移逻辑：[manager.py:L147-L156](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L147-L156)
- 解析逻辑：[manager.py:L160-L187](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L160-L187)

## 保存语义

`save` 使用覆写写入（`"w"`）重建整个 JSONL 文件：

- 第一行写 metadata（含 `last_consolidated`）
- 逐行写 messages
- 最后回写 `_cache[session.key]`

源码锚点：

- [manager.py:L192-L210](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L192-L210)

## 失效与重载

`invalidate` 允许主动丢弃内存缓存，下一次 `get_or_create` 将走磁盘重载路径。

源码锚点：

- [manager.py:L211-L213](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L211-L213)

## `list_sessions` 的轻量读取

会话列表不读取全文件，只读每个 JSONL 第一行 metadata，提高扫描效率。

源码锚点：

- [manager.py:L215-L242](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L215-L242)

## 关键价值

- cache 提供访问性能
- JSONL 提供人类可读性
- 迁移逻辑提供版本升级连续性
- metadata-first 设计支持快速索引
