# Session 模型与存储格式

## 目标

这篇聚焦 `Session` / `SessionManager` 的数据模型设计，以及为什么采用 JSONL。

## Session 核心字段

`Session` 既存对话内容，也存“归纳进度游标”：

- `key`：`channel:chat_id`
- `messages`：append-only 消息数组
- `created_at` / `updated_at`
- `metadata`
- `last_consolidated`

源码锚点：

- [manager.py:L16-L34](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L16-L34)

## append-only 语义

注释明确强调：消息列表不因 memory consolidation 被回写或删改。  
这保证了缓存友好和可追溯性，而归纳状态通过 `last_consolidated` 单独表达。

源码锚点：

- 设计注释：[manager.py:L23-L26](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L23-L26)
- `add_message`：[manager.py:L35-L45](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L35-L45)

## JSONL 文件结构

`save` 时会写成：

1. 第一行 metadata（`_type=metadata`）
2. 后续每行一条 message JSON

这是“流式可读 + 局部鲁棒”的格式，便于简单工具检查与修复。

源码锚点：

- [manager.py:L192-L210](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L192-L210)

## 路径映射

会话 key 经过 `safe_filename(key.replace(":", "_"))` 归一化为文件名，避免非法路径字符。

源码锚点：

- [manager.py:L115-L119](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L115-L119)

## 清理语义

`Session.clear()` 会：

- 清空 messages
- 重置 `last_consolidated = 0`
- 更新时间戳

源码锚点：

- [manager.py:L95-L99](file:///Users/bowhead/nanobot/nanobot/session/manager.py#L95-L99)

相关测试：

- clear 重置 offset：[test_consolidate_offset.py:L78-L86](file:///Users/bowhead/nanobot/tests/test_consolidate_offset.py#L78-L86)
