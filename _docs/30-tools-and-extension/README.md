# 30-工具与扩展

本目录聚焦 NanoBot 的 `tools and execution` 主题：模型如何在运行期拿到工具定义、执行工具、处理失败，并通过扩展机制获得新能力。

## 拆分结论

- 维持 **4 个子目录**，不增不减
- 原因：当前源码正好稳定在 4 条主线，继续细分会打散执行闭环，合并则会损失扩展边界
- 这 4 条主线分别对应：运行时工具协议、内置工具边界、外部 MCP 扩展、技能系统扩展

## 子目录拆分

- `tool-registry-and-runtime`：Tool 基类、ToolRegistry、AgentLoop 工具执行闭环
- `built-in-tools-and-guards`：filesystem/exec/web/message/spawn/cron 的职责与安全边界
- `mcp-external-tools`：MCP 连接、包装命名、enabledTools 过滤、失败与超时处理
- `skills-and-extension-points`：SkillsLoader、上下文注入、workspace 覆盖内置技能

## 文档产出规模

- `tool-registry-and-runtime`：4 篇
- `built-in-tools-and-guards`：4 篇
- `mcp-external-tools`：4 篇
- `skills-and-extension-points`：4 篇
- 合计：16 篇主题文档（不含各目录 README）

## 16 篇规划清单（对照）

### tool-registry-and-runtime

- `tool-registry-and-runtime/01-tool-abstraction-and-schema-contract.md`
- `tool-registry-and-runtime/02-toolregistry-registration-and-dispatch.md`
- `tool-registry-and-runtime/03-agentloop-tool-execution-iteration.md`
- `tool-registry-and-runtime/04-tool-context-injection-and-turn-semantics.md`

### built-in-tools-and-guards

- `built-in-tools-and-guards/01-filesystem-tools-path-and-pagination.md`
- `built-in-tools-and-guards/02-exec-tool-safety-guard-and-timeout.md`
- `built-in-tools-and-guards/03-web-tools-untrusted-content-and-ssrf-guard.md`
- `built-in-tools-and-guards/04-message-spawn-cron-behavior-boundary.md`

### mcp-external-tools

- `mcp-external-tools/01-mcp-connection-lifecycle-and-transport-selection.md`
- `mcp-external-tools/02-mcp-tool-wrapper-timeout-cancel-error.md`
- `mcp-external-tools/03-enabled-tools-filtering-and-name-mapping.md`
- `mcp-external-tools/04-agentloop-mcp-lazy-connect-and-shutdown.md`

### skills-and-extension-points

- `skills-and-extension-points/01-skills-discovery-priority-and-availability.md`
- `skills-and-extension-points/02-skill-metadata-requirements-and-always-load.md`
- `skills-and-extension-points/03-context-injected-skills-summary-and-progressive-load.md`
- `skills-and-extension-points/04-new-capability-onboarding-checklist.md`
