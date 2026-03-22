# 归纳失败降级与恢复路径

## 起点与终点

- 起点：归纳调用失败、返回异常或连续失败次数过多
- 终点：系统不阻塞主链路，并保留可恢复原始信息

## 失败后的基本策略

- 归纳失败不应阻断正常对话
- 记录失败并进入降级路径
- 保证原始历史仍可追溯

源码锚点：

- 归纳逻辑与异常分支：[memory.py](file:///Users/bowhead/nanobot/nanobot/agent/memory.py)
- 会话持久化：[manager.py](file:///Users/bowhead/nanobot/nanobot/session/manager.py)

## 可恢复性保障

- 通过 `HISTORY.md` 保留归纳前后关键内容轨迹
- `MEMORY.md` 作为压缩后的长期语义层
- session 原消息与 consolidator 状态共同构成恢复依据

## 生产建议

- 对连续失败设置告警阈值
- 监控归纳耗时与失败率，提前发现模型/网络波动
- 排障时同时核对 memory 文件与 session 存储
