# 10-主链路

本目录聚焦“用户一条消息进来后发生了什么”。

## 分析范围

- 入口与启动流程
- 消息总线的入站/出站流转
- Agent 从接收请求到生成回复的时序

## 子目录拆分

- `startup-bootstrap`：CLI/Gateway 启动装配、组件 wiring、关闭顺序
- `inbound-routing`：各 channel 如何构造 inbound 事件并进入 AgentLoop
- `agent-execution`：`_process_message` 到 `_run_agent_loop` 的执行闭环
- `outbound-delivery`：outbound 队列、dispatcher、channel send 语义

## 深挖顺序

- 先看 `startup-bootstrap`，建立进程与组件装配全景
- 再看 `inbound-routing`，理解“消息是如何被送进核心”的
- 再看 `agent-execution`，看核心推理/工具闭环与状态写回
- 最后看 `outbound-delivery`，看响应如何被可靠发回外部平台

## 统一分析维度

- 每篇都给出明确起点与终点
- 标注关键状态切换和队列行为
- 区分同步路径与异步路径
- 标注关键异常分支与用户可见结果

## 文档产出规模

- `startup-bootstrap`：4 篇
- `inbound-routing`：4 篇
- `agent-execution`：4 篇
- `outbound-delivery`：4 篇
- 合计：16 篇主题文档（不含各目录 README）
