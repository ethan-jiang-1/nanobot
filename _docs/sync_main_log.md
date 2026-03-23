# _docs 与 main 同步日志

这个文件的目的：对比“上次已应用的 main 基线”和“这次目标 main 基线”，把这段代码 delta 映射为 `_docs` 的修订清单，确保文档始终和源码同步。

## 文件定位（定调）

- 这是 `_docs` 与源码同步的唯一总账文件
- 只记录同步事实与可执行步骤，不写扩展分析长文
- 任何“文档是否已跟上 main 代码”的判断，以本文件为准
- 同步工作必须先更新本文件，再改专题文档
- 若发生争议，按“上次已应用 main 基线 -> 本次目标 main 基线”的窗口重新核对

## 分支定位与约束

- `ethan` 是分析分支，允许有独有 `_docs/*`
- 除 `_docs/*` 外，`ethan` 应尽量贴近 `main`
- 每次同步关注的是：`main_old..main_new` 这段窗口，不是全历史

## 当前同步状态

- 记录时间：2026-03-23
- `_docs` 当前分支/提交：`ethan` / `606b207`
- 上次已应用 main 基线：`未登记（历史补录）`
- 本次目标 main 基线：`20494a2`
- 当前分支与 main 差异计数（`HEAD...main`）：`24 0`

## 同步明细子目录

- 明细目录：`_docs/sync_main_log/`
- 命名规范：`YYYY-MM-DD_main-<OLD>_to_<NEW>.md`
- 特殊值：首次补录允许 `<OLD>=bootstrap`
- 主控文件只保留状态、规则、索引；每次同步细节写入子目录文件

### 明细索引

- [2026-03-23_main-bootstrap_to_20494a2.md](./sync_main_log/2026-03-23_main-bootstrap_to_20494a2.md)

## 核心命令（按窗口）

- 设定窗口：`OLD=<上次main基线>`，`NEW=<本次main基线>`
- 窗口提交：`git log --oneline ${OLD}..${NEW}`
- 窗口文件：`git diff --name-status ${OLD}..${NEW}`
- 窗口代码文件（排除文档）：`git diff --name-only ${OLD}..${NEW} | awk '$1 !~ /^_docs\\// {print}'`
- 当前分支相对 main 概览：`git rev-list --left-right --count HEAD...main`

## 代码到文档映射规则

- `nanobot/agent/loop.py`、`nanobot/cli/commands.py` → `10-main-flow`、`20-agent-core/agent-loop`
- `nanobot/agent/memory.py`、`nanobot/session/*` → `20-agent-core/memory-consolidator`、`20-agent-core/session-manager`
- `nanobot/agent/tools/*`、`nanobot/agent/tools/mcp.py` → `30-tools-and-extension`、`70-special-topics/tool-safety-boundaries`
- `nanobot/channels/*` → `40-channel-layer`、`10-main-flow/outbound-delivery`
- `nanobot/cron/*`、`nanobot/heartbeat/*`、`nanobot/agent/subagent.py` → `50-runtime-services`、`70-special-topics/cancellation-and-shutdown`
- `nanobot/config/*`、`nanobot/providers/*` → `60-config-and-multi-instance`

## 执行流程

1. 先确定 `OLD`（上次已应用 main 基线）和 `NEW`（本次目标 main 基线）
2. 用窗口命令导出 `OLD..NEW` 的提交与文件列表
3. 按映射规则得到“应回看文档目录”
4. 修订 `_docs` 后，记录“已回看/已修订/待修订”
5. 把“上次已应用 main 基线”推进到 `NEW`

## 当前轮摘要

- 详情见明细文件：[2026-03-23_main-bootstrap_to_20494a2.md](./sync_main_log/2026-03-23_main-bootstrap_to_20494a2.md)

## 更新模板

每次同步按以下步骤执行：

1. 在 `_docs/sync_main_log/` 新建文件：`YYYY-MM-DD_main-<OLD>_to_<NEW>.md`
2. 将下面模板写入该新文件并补齐字段
3. 回到本文件“明细索引”追加一行链接

```md
# Sync YYYY-MM-DD · main <OLD> -> <NEW>

- `_docs` 分支/提交：`<branch>` / `<head-sha>`
- 上次已应用 main 基线：`<old-main-sha>`
- 本次目标 main 基线：`<new-main-sha>`
- 窗口差异文件数：`<count>`
- 影响主题目录：`<目录A>`、`<目录B>`、`<目录C>`
- 已完成回看：`<文件或目录清单>`
- 已修订文档：`<文件清单>`
- 待修订文档：`<文件清单>`
```
