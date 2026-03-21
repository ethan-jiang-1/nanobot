# exec 工具的安全守卫与超时边界

## 起点与终点

- 起点：LLM 调用 `exec(command, working_dir?, timeout?)`
- 终点：命令在可控边界内执行并返回输出，或被安全守卫拒绝

## 安全守卫分层

`_guard_command` 在执行前进行多层检查：

1. deny patterns：阻断高危命令模式
2. allow patterns：可选白名单
3. 内网地址检测：阻断对内网/私网的命令访问
4. `restrict_to_workspace`：阻断路径穿越与工作目录外绝对路径

源码锚点：

- 守卫逻辑：[shell.py:L144-L176](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L144-L176)
- 绝对路径提取：[shell.py:L178-L183](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L178-L183)

## 超时与输出控制

- 执行超时上限 `_MAX_TIMEOUT=600`
- 默认输出上限 `_MAX_OUTPUT=10_000`
- 超长输出采用“头尾保留 + 中段截断”，兼顾错误定位与结果概览

源码锚点：

- 参数与上限：[shell.py:L45-L76](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L45-L76)
- 执行与超时处理：[shell.py:L78-L143](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L78-L143)

## 注册开关与运行配置

`AgentLoop` 仅在 `exec_config.enable=True` 时注册 `exec` 工具，并注入 `timeout/path_append/restrict_to_workspace`。

源码锚点：

- exec 注册入口：[loop.py:L123-L130](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L123-L130)
- 配置模型：[schema.py:L118-L143](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L118-L143)
