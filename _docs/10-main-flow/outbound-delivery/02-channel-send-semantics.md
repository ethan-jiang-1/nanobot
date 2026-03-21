# channel 发送语义（以 Telegram/Discord 为例）

## 起点与终点

- 起点：dispatcher 调用 `channel.send(msg)`
- 终点：平台 API 接受消息或返回失败

## Telegram 发送关注点

- 支持普通文本与媒体发送
- 可从 metadata 提取 `message_thread_id` 保持 topic 上下文
- 可结合 `reply_to_message` 策略推断回复线程

相关测试显示 topic/reply 语义被保留：

- progress 消息保留 topic：[test_telegram_channel.py:L310-L326](file:///Users/bowhead/nanobot/tests/test_telegram_channel.py#L310-L326)
- reply 推断 topic 成功：[test_telegram_channel.py:L328-L346](file:///Users/bowhead/nanobot/tests/test_telegram_channel.py#L328-L346)

## Discord 发送前后配合

Discord 入站会记录 `message_id/reply_to/guild_id` 到 metadata，供回发侧决定引用行为与上下文。

源码锚点：

- Discord 入站 metadata：[discord.py:L339-L349](file:///Users/bowhead/nanobot/nanobot/channels/discord.py#L339-L349)

## 抽象层约束

所有具体 channel 都必须实现：

- `start()`：监听外部平台
- `stop()`：释放资源
- `send(msg)`：执行实际发送

源码锚点：

- BaseChannel 抽象接口：[base.py:L52-L77](file:///Users/bowhead/nanobot/nanobot/channels/base.py#L52-L77)

