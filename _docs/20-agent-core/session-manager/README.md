# session-manager

本目录用于拆解 Session 与 SessionManager 的会话模型与持久化策略。

## 本主题要回答的问题

- 会话如何组织与落盘，为什么采用 JSONL
- 历史切片为何要做合法工具调用边界修复
- `last_consolidated` 如何与记忆归纳协作
- cache、磁盘、迁移路径如何协同

## 建议文档拆分

- `01-session-model-and-storage.md`：数据模型与文件格式
- `02-history-slicing-and-tool-boundary.md`：历史裁剪与工具调用合法性
- `03-cache-load-save-and-migration.md`：缓存命中、加载保存、旧路径迁移

## 分析抓手

- `Session.get_history`
- `SessionManager._load`
- `SessionManager.save`
