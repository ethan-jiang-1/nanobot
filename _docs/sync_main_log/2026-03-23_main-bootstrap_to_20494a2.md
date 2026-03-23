# Sync 2026-03-23 · main bootstrap -> 20494a2

## 窗口定义

- `_docs` 分支/提交：`ethan` / `606b207`
- 上次已应用 main 基线：`bootstrap`
- 本次目标 main 基线：`20494a2`
- 当前分支与 main 差异计数（`HEAD...main`）：`24 0`

## 窗口变更主题

- 命令路由与 `/status` 路径调整
- streaming 回包链路与渠道发送语义
- memory consolidation 的 token 预算策略
- MCP schema 归一化兼容处理
- multimodal tool result 注入与回传收敛

## 影响目录映射

- `10-main-flow`、`20-agent-core/agent-loop`
- `10-main-flow/outbound-delivery`、`40-channel-layer`
- `20-agent-core/memory-consolidator`、`70-special-topics/history-and-memory-consistency`
- `30-tools-and-extension/mcp-external-tools`、`30-tools-and-extension/tool-registry-and-runtime`
- `20-agent-core/context-builder`、`30-tools-and-extension/built-in-tools-and-guards`

## 已完成回看

- `_docs/00-overview/README.md`
- `_docs/70-special-topics/tool-safety-boundaries/*`
- `_docs/70-special-topics/README.md`

## 已修订文档

- `_docs/00-overview/README.md`
- `_docs/70-special-topics/README.md`
- `_docs/70-special-topics/tool-safety-boundaries/README.md`
- `_docs/70-special-topics/tool-safety-boundaries/01-capability-surface-and-registration-gates.md`
- `_docs/70-special-topics/tool-safety-boundaries/02-shell-tool-guardrails-and-timeouts.md`
- `_docs/70-special-topics/tool-safety-boundaries/03-web-tool-untrusted-content-and-network-guard.md`
- `_docs/70-special-topics/tool-safety-boundaries/04-side-effect-tools-message-spawn-cron-boundaries.md`

## 待修订文档

- `无（本轮已收敛）`

## 本轮结论

- 本轮完成了主控机制落地：主控文件管规则，明细文件管每次同步细节
- 后续同步只需新增一个同命名规范的明细文件，并在主控索引追加一行
