# 产物写入-执行验证-失败修复的用户可见时序

## 起点与终点

- 起点：用户消息进入 `_process_message`，完成上下文构建
- 终点：返回最终答复，或因 `message` 工具已直发而不再返回普通答复

## 用户视角下的典型链路

当用户目标是“做出可交付产物并验证”时，常见体感顺序是：

1. 先看到进度提示（模型思路摘要 / 工具 hint）
2. 模型调用 `write_file` / `edit_file` 生成或修改文件
3. 模型调用 `exec` 运行构建、测试、脚本或命令
4. 若失败，继续改文件并再次执行
5. 成功后输出最终结果

这条链路并非硬编码流程，而是多轮 tool-calling 闭环自然收敛的结果。

## 系统内部对应时序

### A. 进入执行回路

- `_process_message` 构造 `initial_messages`
- 传入 `_run_agent_loop`

源码锚点：

- 入口与组包：[loop.py:L431-L449](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L431-L449)

### B. 模型发起工具调用

- 每轮带着当前工具定义调用 provider
- 若返回 tool_calls，先记录 assistant 消息，再逐个执行工具

源码锚点：

- 主循环与 tool_calls 分支：[loop.py:L195-L232](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L195-L232)
- 工具定义来源：[registry.py:L34-L35](file:///Users/bowhead/nanobot/nanobot/agent/tools/registry.py#L34-L35)

### C. 工具结果回灌并驱动下一轮

- 每个工具结果都会以 `role=tool` 追加进消息列表
- 下一轮模型直接基于这些真实结果继续决策

源码锚点：

- tool result 追加：[context.py:L173-L179](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L173-L179)
- 迭代继续条件：[loop.py:L206-L246](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L206-L246)

### D. 错误促使“修复-重试”

- ToolRegistry 会把参数错误/执行错误回传给模型
- 额外附带“分析错误后换方案”的提示，促使下一轮修复

源码锚点：

- 错误回传策略：[registry.py:L37-L59](file:///Users/bowhead/nanobot/nanobot/agent/tools/registry.py#L37-L59)

### E. 用户可见进度与收口

- 有 tool_calls 时先发 progress/tool_hint
- 最终结果走普通 outbound；若本轮用了 `message` 工具直发，则主回复抑制

源码锚点：

- progress 发送：[loop.py:L439-L445](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L439-L445)
- message 工具抑制逻辑：[loop.py:L458-L466](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L458-L466)

## 为什么有时“只回答不落盘”

如果任务本质是解释、分析或问答，模型可能在首轮就无 tool_calls，直接输出文本并结束。  
只有当目标需要外部副作用（文件变更、命令执行、外部查询）时，才会进入工具链路。

源码锚点：

- 无 tool_calls 直接收口：[loop.py:L233-L246](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L233-L246)
