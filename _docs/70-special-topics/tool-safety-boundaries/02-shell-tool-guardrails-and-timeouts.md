# shell 工具守卫与超时边界

## 起点与终点

- 起点：模型调用 `exec(command, working_dir?, timeout?)`
- 终点：命令被安全执行并返回结果，或在预检阶段被拒绝

## 守卫链路

`ExecTool` 在真正执行前会跑完整预检：

1. deny pattern 拦截高危命令
2. allow pattern 白名单限制（可选）
3. 内网地址命中拦截
4. `restrict_to_workspace` 限制工作目录与路径范围

源码锚点：

- 执行入口：[shell.py](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py)
- 预检逻辑：[_guard_command](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py)

## 时间与输出上限

- timeout 存在全局上限，避免长时间阻塞
- 输出存在长度上限，避免把超长日志注入模型上下文
- 超长输出采用截断策略，优先保留可定位问题的头尾信息

源码锚点：

- 上限与执行分支：[shell.py](file:///Users/bowhead/nanobot/nanobot/agent/tools/shell.py)

## 线上风险与建议

- 把 `restrict_to_workspace` 视为生产默认值
- deny/allow 规则应按业务场景维护，不要长期空置
- 对失败提示文案保持可读，让模型能继续自修复
