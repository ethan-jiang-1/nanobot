# AgentLoop 的 MCP 延迟连接与关闭回收

## 起点与终点

- 起点：AgentLoop 启动或处理 direct 请求前调用 `_connect_mcp()`
- 终点：MCP 连接可复用，停止时资源被安全回收

## 延迟连接策略

`_connect_mcp` 具备一组防重入条件：

- 已连接则跳过
- 正在连接则跳过
- 无 MCP 配置则跳过

首次连接失败会记录错误并保留后续重试机会（下一次消息再尝试）。

源码锚点：

- 延迟连接与重试语义：[loop.py:L137-L157](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L137-L157)
- run 启动时触发连接：[loop.py:L257-L261](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L257-L261)
- direct 入口触发连接：[loop.py:L513-L515](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L513-L515)

## 关闭阶段回收

`close_mcp` 先等待后台任务清空，再关闭 MCP 连接栈。  
对 MCP SDK 关闭过程中的噪声异常做了容错处理，避免影响主停机流程。

源码锚点：

- close_mcp：[loop.py:L340-L350](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L340-L350)
- 后台任务调度：[loop.py:L352-L356](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L352-L356)

## 设计收益

- 启动阶段不会因慢 MCP server 阻塞整体可用性
- 运行中可对临时故障 server 实现“按消息重试”
- 停机时避免 MCP 连接泄漏
