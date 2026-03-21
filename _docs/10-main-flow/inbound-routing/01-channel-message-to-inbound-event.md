# Channel 事件到 InboundMessage

## 起点与终点

- 起点：Telegram/Discord 等平台回调触发
- 终点：`await bus.publish_inbound(InboundMessage(...))`

## 统一入口：BaseChannel `_handle_message`

所有 channel 最终都调用基类 `_handle_message`：

- 先执行 `is_allowed(sender_id)` 访问控制
- 构造标准化 `InboundMessage`
- 发入 inbound 队列

源码锚点：

- 抽象入口：[base.py:L89-L129](file:///Users/bowhead/nanobot/nanobot/channels/base.py#L89-L129)

## Telegram 实际路径

`_on_message` 或 `_forward_command` 解析平台字段后，统一调用 `_handle_message`，并附加：

- `metadata`（如 message_id）
- `media`（下载后的本地路径）
- `session_key`（topic 场景覆盖）

源码锚点：

- 命令转发：[telegram.py:L686-L699](file:///Users/bowhead/nanobot/nanobot/channels/telegram.py#L686-L699)
- 普通消息转发：[telegram.py:L701-L786](file:///Users/bowhead/nanobot/nanobot/channels/telegram.py#L701-L786)

## Discord 实际路径

`_handle_message_create` 完成内容/附件整理后同样转入 `_handle_message`，并写入 reply/guild metadata。

源码锚点：

- Discord 转发：[discord.py:L288-L349](file:///Users/bowhead/nanobot/nanobot/channels/discord.py#L288-L349)

## 相关测试

- `allow_from` 精确匹配约束：[test_base_channel.py:L21-L25](file:///Users/bowhead/nanobot/tests/test_base_channel.py#L21-L25)
- Telegram topic session key 派生：[test_telegram_channel.py:L274-L282](file:///Users/bowhead/nanobot/tests/test_telegram_channel.py#L274-L282)

