# 进度消息与最终回复的双通道语义

## 起点与终点

- 起点：Agent 在一轮执行中产生中间进度或最终内容
- 终点：前端/渠道按语义展示“过程”与“结果”

## 两类 outbound

- 进度消息：`metadata._progress = True`
- 最终回复：无 `_progress` 标记

如果是进度消息，还会有 `_tool_hint` 区分“工具提示”与“普通进度文本”。

当请求声明 `_wants_stream` 时，还会出现 streaming 元数据：

- `_stream_delta`：本次增量文本
- `_stream_end`：本段流已结束
- `_resuming`：后续是否继续工具轮
- `_streamed`：最终正文已流式输出，渠道侧可跳过重复最终包

源码锚点：

- 进度事件封装：[loop.py:L439-L445](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L439-L445)

## 渠道侧过滤策略

`ChannelManager` 按全局配置过滤进度消息：

- `send_tool_hints = False` 可静默工具提示
- `send_progress = False` 可静默普通进度

源码锚点：

- 过滤逻辑：[manager.py:L124-L128](file:///Users/bowhead/nanobot/nanobot/channels/manager.py#L124-L128)

## CLI 交互侧语义

CLI interactive 模式对进度消息实时渲染，对非进度消息作为回合完成信号。

源码锚点：

- outbound 消费与回合完成：[commands.py:L792-L812](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L792-L812)

## 相关测试

- progress 内容只保留可见文本与 tool hint：[test_message_tool_suppress.py:L89-L116](file:///Users/bowhead/nanobot/tests/test_message_tool_suppress.py#L89-L116)
