# 入站事件模型与会话键派生

## 起点与终点

- 起点：channel 准备发消息进 bus
- 终点：AgentLoop 通过 `msg.session_key` 定位会话

## InboundMessage 字段语义

核心字段：

- `channel`：来源渠道（telegram/discord/...）
- `sender_id`：发送者标识
- `chat_id`：会话容器标识
- `content`：文本内容
- `media`：多媒体本地路径
- `metadata`：渠道自定义字段

源码锚点：

- 模型定义：[events.py:L8-L25](file:///Users/bowhead/nanobot/nanobot/bus/events.py#L8-L25)

## session key 规则

- 默认：`f"{channel}:{chat_id}"`
- 覆盖：`session_key_override` 存在时优先使用

这使 thread/topic 场景可以把同一 chat 拆成独立会话。

源码锚点：

- `session_key` 属性：[events.py:L21-L24](file:///Users/bowhead/nanobot/nanobot/bus/events.py#L21-L24)
- BaseChannel 透传 override：[base.py:L119-L127](file:///Users/bowhead/nanobot/nanobot/channels/base.py#L119-L127)

## 队列边界

`publish_inbound` 只做入队，不做业务逻辑；消费逻辑统一在 AgentLoop。

源码锚点：

- inbound publish/consume：[queue.py:L20-L26](file:///Users/bowhead/nanobot/nanobot/bus/queue.py#L20-L26)

## 相关测试

- topic key 派生断言：[test_telegram_channel.py:L274-L282](file:///Users/bowhead/nanobot/tests/test_telegram_channel.py#L274-L282)

