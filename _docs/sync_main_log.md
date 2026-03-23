# _docs 与 main 同步日志

这个文件只做一件事：记录 `_docs` 对齐的 main 基线，以及每次同步要关注的代码 delta。

## 当前基线

- 记录时间：2026-03-23
- `_docs` 当前分支：`ethan`
- `_docs` 当前提交：`606b207`
- 对齐 main 提交：`20494a2`
- 差异计数（`HEAD...main`）：`24 0`（左侧=当前分支独有，右侧=main 独有）

## 固定核对命令

- 提交差异：`git log --oneline --left-right --cherry-pick main...HEAD`
- 文件差异：`git diff --name-status main...HEAD`
- 仅代码差异（排除 `_docs`）：`git diff --name-status main...HEAD | awk '$2 !~ /^_docs\\// {print}'`
- main 最近 20 提交改动文件：`git log --pretty=format:'--- %h %s' --name-only main~20..main`

## 主题映射规则

- `nanobot/agent/loop.py`、`nanobot/cli/commands.py` → `10-main-flow`、`20-agent-core/agent-loop`
- `nanobot/agent/memory.py`、`nanobot/session/*` → `20-agent-core/memory-consolidator`、`20-agent-core/session-manager`
- `nanobot/agent/tools/*`、`nanobot/agent/tools/mcp.py` → `30-tools-and-extension`、`70-special-topics/tool-safety-boundaries`
- `nanobot/channels/*` → `40-channel-layer`、`10-main-flow/outbound-delivery`
- `nanobot/cron/*`、`nanobot/heartbeat/*`、`nanobot/agent/subagent.py` → `50-runtime-services`、`70-special-topics/cancellation-and-shutdown`
- `nanobot/config/*`、`nanobot/providers/*` → `60-config-and-multi-instance`

## 本轮 main 高影响点

- `20494a2` 命令路由重构：回看 `/status` 与主链路命令短路文档
- `aba0b83` memory 归纳留白：回看 token 压力与归纳触发策略
- `bd621df` `e79b9f4` `f2e1cb3` `9d5e511` streaming 链路：回看 outbound 与渠道发送语义
- `e87bb0a` `b6cf702` MCP schema 归一化：回看 MCP 工具包装与 schema 约束
- `445a96a` multimodal tool result 收敛：回看 context 组装与工具结果注入路径

## 更新模板

复制下面块并追加到文件末尾：

```md
## Sync YYYY-MM-DD

- `_docs` 分支/提交：`<branch>` / `<head-sha>`
- 对齐 main 提交：`<main-sha>`
- 差异计数（`HEAD...main`）：`<left> <right>`
- 影响主题：`<目录A>`、`<目录B>`、`<目录C>`
- 已完成回看：`<文件或目录清单>`
- 待修订文档：`<文件清单>`
```
