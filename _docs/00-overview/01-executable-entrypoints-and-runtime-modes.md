# NanoBot 可执行入口与运行模式总览（开发/运维视角）

## 起点与终点

- 起点：明确 NanoBot 有哪些可执行入口、每个入口跑多久、要不要重启
- 终点：形成“入口总览 + 分篇细化”的运维与开发共用手册

## 一句话结论

- 安装后外部主入口只有一个：`nanobot`
- 复杂入口主要是 `gateway` 与 `agent`，已拆成独立专题并补了 Mermaid
- 当前没有独立 `doctor` 命令，诊断职责由 `status`、`channels status`、`/status` 组合承担

源码锚点：

- 二进制脚本声明：[pyproject.toml:L74-L75](file:///Users/bowhead/nanobot/pyproject.toml#L74-L75)
- 模块启动入口（`python -m nanobot`）：[__main__.py:L1-L8](file:///Users/bowhead/nanobot/nanobot/__main__.py#L1-L8)

## 分篇结构

- 复杂长驻入口（daemon）：[02-gateway-entrypoint-lifecycle.md](file:///Users/bowhead/nanobot/_docs/00-overview/02-gateway-entrypoint-lifecycle.md)
- 复杂交互入口（CLI agent）：[03-agent-entrypoint-modes-and-flow.md](file:///Users/bowhead/nanobot/_docs/00-overview/03-agent-entrypoint-modes-and-flow.md)
- 短命令入口与诊断面：[04-onboard-status-and-ops-entrypoints.md](file:///Users/bowhead/nanobot/_docs/00-overview/04-onboard-status-and-ops-entrypoints.md)
- 会话内可执行命令（slash）：[05-in-session-slash-entrypoints.md](file:///Users/bowhead/nanobot/_docs/00-overview/05-in-session-slash-entrypoints.md)

## 入口全景（按执行层级）

| 层级 | 入口 | 典型命令 | 生命周期 | 复杂度 |
|---|---|---|---|---|
| 进程入口 | packaging script | `nanobot ...` | 取决于子命令 | 低 |
| 进程入口 | Python module | `python -m nanobot ...` | 取决于子命令 | 低 |
| CLI 子命令 | `gateway` | `nanobot gateway` | 长驻 | 高 |
| CLI 子命令 | `agent` | `nanobot agent` | 短时或长会话 | 高 |
| CLI 子命令 | `onboard` | `nanobot onboard` | 短时 | 中 |
| CLI 子命令 | `status` | `nanobot status` | 短时 | 低 |
| CLI 子命令组 | `channels` | `nanobot channels status/login` | 短时 | 中 |
| CLI 子命令组 | `plugins` | `nanobot plugins list` | 短时 | 低 |
| CLI 子命令组 | `provider` | `nanobot provider login ...` | 短时 | 中 |
| 会话内入口 | slash 命令 | `/stop /restart /status /new /help` | 运行中即时生效 | 中 |

## 重启 vs 动态生效矩阵

| 变更项 | 是否需要重启 `gateway` | 原因 |
|---|---|---|
| `gateway.port` | 需要 | 端口在 gateway 启动时读取并绑定 |
| `channels.*.enabled` / 渠道配置 | 需要 | `ChannelManager` 在启动时实例化渠道集合 |
| `agents.defaults.model` | 需要 | `AgentLoop`/provider 在启动时构造，运行中不热替换 |
| `tools.exec`、`tools.restrict_to_workspace`、`tools.mcp_servers` | 需要 | 工具注册与 MCP 连接发生在 AgentLoop 生命周期内 |
| `gateway.heartbeat.enabled` / `interval_s` | 需要 | `HeartbeatService` 由启动配置构造，运行中无配置热加载 |
| `HEARTBEAT.md` 内容 | 不需要 | 心跳 tick 时实时读取文件内容 |
| cron `jobs.json` 任务文件（外部修改） | 不需要（通常） | `CronService` 检测文件 mtime 变化并重载 |
| 当前进程异常或配置大改后想就地重启 | 可用 `/restart` | 通过 `os.execv` 原地拉起同一进程 |

源码锚点：

- 运行时配置加载：[commands.py:L445-L462](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L445-L462)
- ChannelManager 构造位置：[commands.py:L588-L589](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L588-L589)
- AgentLoop 构造位置：[commands.py:L523-L538](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L523-L538)
- HeartbeatService 构造位置：[commands.py:L632-L641](file:///Users/bowhead/nanobot/nanobot/cli/commands.py#L632-L641)
- heartbeat 实时读取文件：[service.py:L77-L83](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L77-L83), [service.py:L147-L150](file:///Users/bowhead/nanobot/nanobot/heartbeat/service.py#L147-L150)
- cron 外部修改自动重载：[service.py:L80-L89](file:///Users/bowhead/nanobot/nanobot/cron/service.py#L80-L89)
- `/restart` 原地重启：[builtin.py:L32-L41](file:///Users/bowhead/nanobot/nanobot/command/builtin.py#L32-L41)

## 快速使用顺序

1. 先读 gateway 专题，建立 daemon 心智模型
2. 再读 agent 专题，理解交互模式与会话执行链
3. 再读短命令专题，明确初始化、巡检、登录运维动作
4. 最后读 slash 专题，掌握运行中控制与重启语义
