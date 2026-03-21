# outbound-delivery

本目录聚焦“回复如何发出去”：从 outbound 队列，到 ChannelManager 分发，再到各平台 send 细节。

## 本主题要回答的问题

- outbound 消息如何从 agent 进入队列
- dispatcher 如何按 channel 路由并过滤进度消息
- channel 的 send 如何处理回复线程与多媒体
- 发送失败时系统如何退化

## 建议文档拆分

- `01-outbound-queue-and-dispatcher.md`：队列消费与分发主循环
- `02-channel-send-semantics.md`：Telegram/Discord 等 send 语义
- `03-progress-message-vs-final-response.md`：进度消息与最终回复的区别
- `04-failure-path-retry-and-user-visible-effects.md`：失败分支与可见效果

## 产出数量

- 预计 4 篇，覆盖“agent 输出 -> 外部平台送达”链路

