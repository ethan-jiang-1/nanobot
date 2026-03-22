# message / spawn / cron 的副作用边界

## 起点与终点

- 起点：模型调用会触达外部系统或后台执行的工具
- 终点：动作被执行且可追踪，不破坏主会话可控性

## message：外发语义与回合抑制

- `message` 可向指定会话投递
- 当成功投递到默认目标，会标记“本回合已主动发送”
- 主循环据此抑制重复最终回包

源码锚点：

- 工具实现：[message.py](file:///Users/bowhead/nanobot/nanobot/agent/tools/message.py)
- 抑制判断：[loop.py](file:///Users/bowhead/nanobot/nanobot/agent/loop.py)

## spawn：后台任务交接

- `spawn` 只负责创建任务与保存来源上下文
- 真正执行由 `SubagentManager` 接管
- 结果通过 system 通道异步回注主链路

源码锚点：

- spawn 工具：[spawn.py](file:///Users/bowhead/nanobot/nanobot/agent/tools/spawn.py)
- 子代理运行时：[subagent.py](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py)

## cron：延迟/周期执行与递归保护

- `cron` 支持 add/list/remove 生命周期
- 回调执行与普通会话隔离
- 明确禁止 cron 回调里继续创建 cron，避免递归调度

源码锚点：

- 工具实现：[cron.py](file:///Users/bowhead/nanobot/nanobot/agent/tools/cron.py)
- 服务执行：[service.py](file:///Users/bowhead/nanobot/nanobot/cron/service.py)
