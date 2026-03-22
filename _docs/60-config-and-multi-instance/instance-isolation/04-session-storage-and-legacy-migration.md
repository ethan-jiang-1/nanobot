# 会话存储与 legacy 迁移

## 起点与终点

- 起点：SessionManager 读取某个会话 key
- 终点：会话文件位于当前 workspace/sessions，必要时从 legacy 路径迁移

## 目录结构

- 当前会话目录：`workspace/sessions/*.jsonl`
- legacy 目录：`~/.nanobot/sessions/*.jsonl`
- key 会先做安全文件名转换，避免非法字符

源码锚点：

- sessions 目录初始化：[manager.py:L109-L113](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L109-L113)
- 路径生成：[manager.py:L115-L123](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L115-L123)

## 迁移触发条件

- 读取当前路径不存在时，检查 legacy 路径
- 若 legacy 文件存在，尝试 `shutil.move` 到当前路径
- 迁移成功后继续按新路径读取 JSONL

源码锚点：

- 迁移分支：[manager.py:L145-L156](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L145-L156)

## JSONL 读取与保存语义

- 第一行 metadata，后续行 messages
- `save` 以覆写方式重建整个文件
- `list_sessions` 只读第一行以提升扫描效率

源码锚点：

- 加载解析：[manager.py:L160-L187](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L160-L187)
- 保存流程：[manager.py:L192-L210](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L192-L210)
- 列表扫描：[manager.py:L215-L242](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L215-L242)

## 边界提醒

- 会话隔离主要由 workspace 决定，而非 config data_dir
- 因此多实例若共享 workspace，sessions 仍会共享
