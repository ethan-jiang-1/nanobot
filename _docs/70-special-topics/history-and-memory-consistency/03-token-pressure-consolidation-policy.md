# token 压力下的归纳策略

## 起点与终点

- 起点：会话 token 接近上限
- 终点：通过归纳释放上下文预算，同时维持对话语义连续

## 双阶段策略

- preflight：调用模型前先尝试归纳，防止本轮直接爆窗
- post-turn：回合结束后异步归纳，为下一轮预留空间

源码锚点：

- 主链路触发点：[loop.py](file:///Users/bowhead/nanobot/nanobot/agent/loop.py)
- 归纳器实现：[memory.py](file:///Users/bowhead/nanobot/nanobot/agent/memory.py)

## 边界选择原则

- 从 `last_consolidated` 开始扫描
- 只在 user turn 边界切分
- 优先满足目标 token 回收量

这样可以避免在 assistant/tool 中间硬切造成语义破损。

## 关键状态

- `last_consolidated` 是逻辑游标，不是物理删历史
- 原始消息仍留在 session，可用于回放与审计
