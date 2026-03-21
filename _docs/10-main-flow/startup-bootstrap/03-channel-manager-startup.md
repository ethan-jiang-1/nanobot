# ChannelManager 启动与分发协同

## 起点与终点

- 起点：`channels = ChannelManager(config, bus)`
- 终点：`start_all` 拉起 outbound dispatcher 与所有启用 channel

## 初始化阶段

`ChannelManager` 初始化时会：

- 通过 `discover_all()` 收集内建与插件 channel
- 按配置筛选 enabled channel
- 实例化 channel 并注入同一个 `MessageBus`
- 校验 `allow_from`，空数组直接阻断启动

源码锚点：

- 初始化与校验：[manager.py:L25-L67](file:///Users/bowhead/nanobot/nanobot/channels/manager.py#L25-L67)

## 运行阶段

`start_all` 顺序：

1. 启动 `_dispatch_outbound` 后台任务
2. 并发启动所有 channel `start()`
3. 等待这些长任务常驻

源码锚点：

- `start_all`：[manager.py:L75-L92](file:///Users/bowhead/nanobot/nanobot/channels/manager.py#L75-L92)

## 为什么先拉起 outbound dispatcher

这样 agent 在早期阶段发布的 outbound 消息不会无消费者积压，避免“首条响应丢在队列里没人发”。

## 相关测试

- 插件 channel 可被初始化：[test_channel_plugins.py:L159-L184](file:///Users/bowhead/nanobot/tests/test_channel_plugins.py#L159-L184)
- disabled 插件会被跳过：[test_channel_plugins.py:L186-L207](file:///Users/bowhead/nanobot/tests/test_channel_plugins.py#L186-L207)

