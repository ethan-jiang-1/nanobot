# built-in-tools-and-guards

本目录聚焦内置工具集合与执行边界：filesystem、exec、web、message、spawn、cron。

## 本主题要回答的问题

- 内置工具覆盖了哪些能力面
- 读写文件和命令执行的安全边界在哪里
- web 工具如何处理不可信外部内容
- message/spawn/cron 这类“执行型工具”如何影响主流程

## 建议文档拆分

- `01-filesystem-tools-path-and-pagination.md`
- `02-exec-tool-safety-guard-and-timeout.md`
- `03-web-tools-untrusted-content-and-ssrf-guard.md`
- `04-message-spawn-cron-behavior-boundary.md`
