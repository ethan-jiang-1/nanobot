# shell 工具守卫与超时边界

## 起点与终点

- 起点：模型调用 `exec(command, working_dir?, timeout?)`
- 终点：命令被安全执行并返回结果，或在预检阶段被拒绝

## 执行前守卫链路

`ExecTool` 在真正执行前会跑完整预检：

1. deny patterns：拦截破坏性命令（如 `rm -rf`、`mkfs`、`dd if=`、`shutdown`）
2. allow patterns：若配置了白名单，只允许匹配命令执行
3. 内网 URL 检测：阻断命令中对私网/metadata 地址的访问
4. workspace 限制：阻断 `../` 穿越与工作目录外绝对路径

源码锚点：

- 预检入口：[shell.py:L83-L85](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L83-L85)
- deny/allow/内网/路径守卫：[shell.py:L144-L176](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L144-L176)
- 绝对路径提取：[shell.py:L178-L183](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L178-L183)
- 内网 URL 检测：[network.py:L97-L104](file:///Users/bowhead/nanobot/nanobot/security/network.py#L97-L104)

## 执行中资源边界

- timeout 有硬上限 `_MAX_TIMEOUT=600`，防止“永久占用”
- 超时后会 kill 子进程，避免僵尸任务残留
- 输出有硬上限 `_MAX_OUTPUT=10_000`，防止日志灌爆上下文
- 超长输出采用头尾保留，兼顾错误定位与结果摘要

源码锚点：

- timeout/输出上限：[shell.py:L45-L47](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L45-L47)
- 超时处理：[shell.py:L102-L114](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L102-L114)
- 截断策略：[shell.py:L129-L137](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py#L129-L137)

## 当前能力边界判断

- 强项：对明显危险命令、SSRF 式命令和路径越权有直接拦截
- 不足：仍是正则级别守卫，不等于完整沙箱或 syscall 级隔离
- 结论：这是一套实用“护栏”，不是完全隔离“监狱”

## 关键测试覆盖

- 命令内网 URL 拦截：[test_exec_security.py:L26-L69](file:///Users/bowhead/nanobot/tests/test_exec_security.py#L26-L69)
- 公网 URL 放行：[test_exec_security.py:L53-L58](file:///Users/bowhead/nanobot/tests/test_exec_security.py#L53-L58)
