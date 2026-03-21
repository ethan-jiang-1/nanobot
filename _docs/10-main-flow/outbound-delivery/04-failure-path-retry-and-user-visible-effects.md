# 发送失败路径与用户可见影响

## 起点与终点

- 起点：某次 outbound 发送出现异常或目标不可用
- 终点：系统记录错误并继续主循环，不拖垮全局

## dispatcher 层失败处理

`_dispatch_outbound` 对发送失败采用“记录并继续”：

- `channel.send(msg)` 抛错 -> error log
- unknown channel -> warning log
- dispatcher 不退出，继续消费下一条

源码锚点：

- 失败处理：[manager.py:L130-L142](file:///Users/bowhead/nanobot/nanobot/channels/manager.py#L130-L142)

## run 层失败处理

`_dispatch` 若 `_process_message` 抛异常，会回送统一错误文案给用户，保证请求有可见结果。

源码锚点：

- `_dispatch` 异常兜底：[loop.py:L333-L338](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L333-L338)

## 平台发送重试（示例）

Telegram 文本发送存在 timeout/backoff 重试逻辑，最终仍失败时不会抛穿主链路。

相关测试：

- timeout 重试路径：[test_telegram_channel.py:L250-L272](file:///Users/bowhead/nanobot/tests/test_telegram_channel.py#L250-L272)

## 用户可见影响总结

- 单条发送失败通常表现为“这条消息没送达”，而不是整个机器人停摆
- 业务执行失败通常会收到 `Sorry, I encountered an error.`

