# message / spawn / cron 的副作用边界

## 起点与终点

- 起点：模型调用会触达外部系统或后台执行的工具
- 终点：动作被执行且可追踪，不破坏主会话可控性

## message：外发语义与回合收敛

- `message` 可向指定会话投递
- 当成功投递到默认目标，会标记“本回合已主动发送”
- 主循环据此抑制重复最终回包，避免“一次回答发两遍”

源码锚点：

- 执行与 `_sent_in_turn`：[message.py:L73-L107](file:///Users/bowhead/nanobot/nanobot/agent/tools/message.py#L73-L107)
- 上下文注入：[loop.py:L159-L165](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L159-L165)

## spawn：后台执行与能力沙箱

- `spawn` 只负责创建任务与保存来源上下文
- 真正执行由 `SubagentManager` 接管
- 子代理默认不注册 `message/spawn/cron`，避免递归副作用扩散
- 结果通过 `system` 通道异步回注主链路

源码锚点：

- spawn 工具：[spawn.py](file:///Users/bowhead/nanobot/nanobot/agent/tools/spawn.py)
- 子代理受限工具集：[subagent.py:L93-L109](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L93-L109)
- system 回注路径：[subagent.py:L189-L198](file:///Users/bowhead/nanobot/nanobot/agent/subagent.py#L189-L198)

## cron：延迟/周期执行与递归保护

- `cron` 支持 add/list/remove 生命周期
- 回调执行与普通会话隔离
- 明确禁止 cron 回调里继续创建 cron，避免递归调度

源码锚点：

- 递归防护判定：[cron.py:L85-L88](file:///Users/bowhead/nanobot/nanobot/agent/tools/cron.py#L85-L88)
- 会话上下文要求：[cron.py:L103-L107](file:///Users/bowhead/nanobot/nanobot/agent/tools/cron.py#L103-L107)
- 回调上下文包裹：[commands.py:L560-L573](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L560-L573)

## 外发消息前置安全：谁能触发这些副作用

副作用工具的前提是“消息先能进主循环”，而入口有接入层 ACL：

- 各 channel 的 `allow_from` 默认为空，空列表即拒绝全部
- 只有命中白名单（或显式 `*`）的 sender 才能触发工具链路
- `sender_id` 必须精确匹配，避免拼接绕过

源码锚点：

- 通道白名单判定：[base.py:L79-L87](file:///Users/bowhead/nanobot/nanobot/channels/base.py#L79-L87)
- 白名单单测（精确匹配）：[test_base_channel.py:L21-L25](file:///Users/bowhead/nanobot/tests/test_base_channel.py#L21-L25)

## WhatsApp Bridge 边界

- Node bridge 仅绑定 `127.0.0.1`，不对公网监听
- 可配置 `BRIDGE_TOKEN` 强制首包认证，未认证连接会被关闭

源码锚点：

- localhost 绑定与 token 鉴权：[server.ts:L27-L59](file:///Users/bowhead/nanobot/bridge/src/server.ts#L27-L59)
- Python 侧 token 握手：[whatsapp.py:L61-L64](file:///Users/bowhead/nanobot/nanobot/channels/whatsapp.py#L61-L64)
