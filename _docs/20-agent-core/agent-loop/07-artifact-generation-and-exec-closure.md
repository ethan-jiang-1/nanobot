# 产物落盘与执行闭环机制

## 起点与终点

- 起点：`_process_message()` 组装完 `initial_messages` 后进入 `_run_agent_loop()`
- 终点：模型给出最终 assistant 文本，或达到最大迭代次数退出

## 机制不是“写文件特判”，而是“函数调用闭环”

AgentLoop 并没有写“遇到某类任务就强制 `write_file`”的硬编码分支。  
它做的是把工具定义交给模型，再把工具执行结果回灌到同一轮上下文，循环直到收敛。

源码锚点：

- 进入迭代与工具定义注入：[loop.py:L184-L204](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L184-L204)
- assistant tool_call 追加与工具执行：[loop.py:L215-L232](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L215-L232)
- 工具结果作为 `role=tool` 回灌：[context.py:L173-L179](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L173-L179)
- 终止条件（无 tool_call 或超轮次）：[loop.py:L233-L255](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L233-L255)

## 为什么会出现“先写文件，再跑命令”

典型链路是：

1. 模型先调用 `write_file` / `edit_file` 生成或修改产物  
2. 读取工具返回（成功/失败）  
3. 再调用 `exec` 做构建、测试、运行验证  
4. 根据 `exec` 输出继续修正或结束

这个行为来自回路结构本身，不依赖特定模型品牌，也不依赖单一提示词模板。

## 会话写回如何保留这条链路

每轮新增消息会写入 session，工具输出过长会被截断保留头部。  
因此“写文件→执行→修复”的主要证据会在后续轮次继续可见。

源码锚点：

- turn 写回与截断：[loop.py:L468-L503](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L468-L503)

## 可观测性与用户感知

当模型决定调用工具时，AgentLoop 会发进度提示（包括工具 hint），所以用户会感知到“正在写文件/正在执行命令”的中间阶段，而不是只看到最终回答。

源码锚点：

- 工具 hint 与进度上报：[loop.py:L206-L214](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L206-L214)
- 总线 progress 事件：[loop.py:L439-L449](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L439-L449)
