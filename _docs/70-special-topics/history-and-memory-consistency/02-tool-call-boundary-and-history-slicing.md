# tool 调用边界与历史切片合法性

## 起点与终点

- 起点：会话历史超过窗口，需要按最近消息切片
- 终点：切片结果仍满足“assistant tool_call 与 tool 响应成对”

## 核心约束

如果在 assistant tool_call 与 tool 响应之间硬切，会出现孤儿消息，导致模型上下文非法或语义断裂。

`Session.get_history` 会在切片后做边界修复，移除不完整片段，保证结构合法。

源码锚点：

- 会话模型与切片：[manager.py](file:///Users/bowhead/nanobot/nanobot/session/manager.py)

## 生产价值

- 降低 provider 对非法消息序列的拒绝概率
- 减少“模型突然不认工具上下文”的偶发故障
- 提高长会话稳定性

## 排障抓手

- 先看当前窗口起始是否落在 tool group 中间
- 再看修复后是否仍保留关键用户问题上下文
