# runtime context 剥离与历史卫生

## 起点与终点

- 起点：`ContextBuilder` 在本轮输入中注入 runtime 元数据
- 终点：`_save_turn` 写会话时只保留长期有价值内容

## 为什么必须剥离

runtime 内容（时间、channel、chat_id）服务于“本轮推理”，若直接持久化会导致：

- 历史噪声持续膨胀
- 下轮推理受到过期上下文污染
- 历史可读性显著下降

源码锚点：

- runtime 构建：[context.py](file:///Users/bowhead/nanobot/nanobot/agent/context.py)
- 回写清洗：[_save_turn](file:///Users/bowhead/nanobot/nanobot/agent/loop.py)

## `_save_turn` 其他卫生策略

- 跳过空 assistant 消息
- 截断超长 tool 输出
- 多模态图片转稳定占位文本

## 结果

- 历史可长期复用
- 会话大小增长更可控
- 后续归纳与排障更稳定
