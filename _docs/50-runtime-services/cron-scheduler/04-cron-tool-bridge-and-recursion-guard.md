# Cron 工具桥接与递归防护

## 起点与终点

- 起点：模型调用 `cron(action=...)`
- 终点：任务被创建/列出/删除，且回调执行链路可控

## Tool 参数契约

`CronTool` 对外只暴露 `add/list/remove`，并给出三类时间表达：

- `every_seconds`
- `cron_expr` + 可选 `tz`
- `at`（ISO datetime）

源码锚点：

- 工具定义与参数：[cron.py:L34-L72](file:///Users/bowhead/nanobot/nanobot/agent/tools/cron.py#L34-L72)
- action 分发：[cron.py:L74-L93](file:///Users/bowhead/nanobot/nanobot/agent/tools/cron.py#L74-L93)

## 会话上下文桥接

`CronTool` 依赖 `set_context(channel, chat_id)` 注入当前会话路由：

- add 时自动把 `deliver/channel/to` 写入 payload
- 后续 job 执行可据此把结果回发原会话

源码锚点：

- 上下文注入：[loop.py:L159-L165](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L159-L165)
- add 组包：[cron.py:L95-L145](file:///Users/bowhead/nanobot/nanobot/agent/tools/cron.py#L95-L145)

## 递归调度防护

cron 回调执行期间，gateway 会把 `CronTool` 标记为“cron 上下文”：

- 进入回调前 `set_cron_context(True)`
- 回调结束后恢复 token
- 若此时模型再尝试 `cron add`，工具直接报错拒绝

这避免“定时任务再创建定时任务”的无限扩散。

源码锚点：

- 回调上下文包裹：[commands.py:L560-L573](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L560-L573)
- 防护判定：[cron.py:L85-L88](file:///Users/bowhead/nanobot/nanobot/agent/tools/cron.py#L85-L88)

## 与主链路的交汇点

CronService 只负责“到点执行”，真正任务处理走 `agent.process_direct`：

- 构造 reminder note 作为输入
- 使用 `session_key=f"cron:{job.id}"` 隔离会话
- 若启用 deliver，再走评估后 outbound 投递

源码锚点：

- on_job 主逻辑：[commands.py:L547-L590](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L547-L590)
