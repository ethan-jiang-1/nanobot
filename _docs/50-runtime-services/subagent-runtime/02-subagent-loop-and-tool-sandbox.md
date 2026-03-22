# 子代理循环与工具沙箱

## 起点与终点

- 起点：`_run_subagent` 开始执行后台任务
- 终点：子代理产出最终文本或失败文本

## 子代理工具集

子代理工具集是“受限默认集”，明确不注册 message/spawn/cron：

- 文件工具：read/write/edit/list
- shell 工具：exec（受执行配置约束）
- web 工具：search/fetch

这让子代理专注执行，不直接触发外部投递或再次分叉任务。

源码锚点：

- 工具注册段：[subagent.py:L93-L109](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L93-L109)

## workspace 约束

当 `restrict_to_workspace=True` 时：

- 文件工具限制在 workspace 下
- 额外允许只读 builtin skills 目录
- exec 也受 workspace 边界约束

源码锚点：

- allowed_dir 与 extra_read：[subagent.py:L95-L100](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L95-L100)
- exec 限制：[subagent.py:L101-L106](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L101-L106)

## 子代理迭代闭环

执行循环与主代理类似，但有独立上限：

- 初始化 system+user 两条消息
- 最多 15 轮 tool-calling 迭代
- 有 tool call 时执行工具并回填 tool 消息
- 无 tool call 时收敛为 final result

源码锚点：

- 消息初始化：[subagent.py:L110-L115](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L110-L115)
- 迭代主循环：[subagent.py:L116-L155](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L116-L155)

## Prompt 设计点

子代理 system prompt 明确强调：

- 只处理分配任务
- Web 内容不可信
- 结果会回报主代理

源码锚点：

- prompt 构建：[subagent.py:L200-L221](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L200-L221)
