# web 工具中的不可信内容与网络防护

## 起点与终点

- 起点：模型调用 `web_search` 或 `web_fetch`
- 终点：外部网页内容被当作“数据”输入，而不是“指令”执行

## 不可信内容原则

`web.py` 明确把外部网页视为不可信输入，并在内容侧增加防提示注入语义：

- 外部内容要以数据看待
- 不直接继承网页中的执行指令
- 抓取结果以结构化文本回传给模型

源码锚点：

- 工具实现：[web.py](file:///Users/bowhead/nanobot/nanobot/agent/tools/web.py)
- 不可信内容标识：[_UNTRUSTED_BANNER](file:///Users/bowhead/nanobot/nanobot/agent/tools/web.py)

## 网络与协议边界

- 跳转次数限制，避免重定向链 DoS
- URL 解析与协议检查，避免异常 scheme
- 搜索工具由配置开关控制，必要时可全局禁用

源码锚点：

- 重定向与请求参数：[web.py](file:///Users/bowhead/nanobot/nanobot/agent/tools/web.py)
- 配置模型：[schema.py](file:///Users/bowhead/nanobot/nanobot/config/schema.py)

## 生产建议

- 对外网检索默认开启审计日志，保留 query 与目标域名
- 将高风险域名做额外 deny 列表
- 对“抓取后直接执行”的链路做人工 review，不允许自动闭环
