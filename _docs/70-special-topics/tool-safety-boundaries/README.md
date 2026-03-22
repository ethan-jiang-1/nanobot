# tool-safety-boundaries

本目录聚焦工具层的“能力边界与副作用控制”：哪些工具会触达外部世界、如何限制风险、失败后如何保持可恢复。

## 本主题要回答的问题

- 哪些工具默认注册，哪些受配置开关控制
- 文件、命令、网络访问分别有哪些硬边界
- `message/spawn/cron` 这类执行型工具如何避免越权副作用
- 工具失败时主链路如何继续推进并给出可解释结果

## 建议文档拆分

- `01-capability-surface-and-registration-gates.md`：工具面与注册闸门
- `02-shell-tool-guardrails-and-timeouts.md`：命令执行守卫与超时
- `03-web-tool-untrusted-content-and-network-guard.md`：外部内容不可信与网络防护
- `04-side-effect-tools-message-spawn-cron-boundaries.md`：执行型工具的行为边界

## 分析抓手

- `AgentLoop._register_default_tools`
- `ExecTool.execute` / `_guard_command`
- `WebFetchTool.execute` / `WebSearchTool.execute`
- `MessageTool.execute` / `SpawnTool.execute` / `CronTool.execute`
