# tool-safety-boundaries

本目录聚焦工具层的“能力边界与副作用控制”：哪些工具会触达外部世界、如何限制风险、失败后如何保持可恢复。

## 安全总览结论

从代码现状看，NanoBot 的安全模型属于“默认护栏型”：

- 入口侧有白名单（`allow_from`）控制谁能触发链路
- 工具侧有命令守卫、路径边界、SSRF 防护、超时和输出上限
- 副作用工具有上下文约束，子代理有能力沙箱
- 外部扩展（MCP/bridge）有最小暴露和连接鉴权选项

它的定位是“工程可用的风险收敛”，不是“强隔离零信任沙箱”。

## 本主题要回答的问题

- 哪些工具默认注册，哪些受配置开关控制
- 文件、命令、网络访问分别有哪些硬边界
- `message/spawn/cron` 这类执行型工具如何避免越权副作用
- 工具失败时主链路如何继续推进并给出可解释结果

## 阅读顺序

- 先看 `01`，理解“能力面如何被收口”
- 再看 `02/03`，理解命令与网络两条高风险主线
- 最后看 `04`，理解副作用动作如何避免失控

## 文档拆分

- `01-capability-surface-and-registration-gates.md`：工具面与注册闸门
- `02-shell-tool-guardrails-and-timeouts.md`：命令执行守卫与超时
- `03-web-tool-untrusted-content-and-network-guard.md`：外部内容不可信与网络防护
- `04-side-effect-tools-message-spawn-cron-boundaries.md`：执行型工具的行为边界

## 分析抓手

- `AgentLoop._register_default_tools`
- `ExecTool.execute` / `_guard_command`
- `WebFetchTool.execute` / `WebSearchTool.execute`
- `MessageTool.execute` / `SpawnTool.execute` / `CronTool.execute`

## 覆盖与盲区

- 覆盖：路径穿越、私网访问、明显危险命令、调度递归、副作用路由
- 盲区：内建 rate limit、系统级沙箱、密钥密文存储、细粒度审计
