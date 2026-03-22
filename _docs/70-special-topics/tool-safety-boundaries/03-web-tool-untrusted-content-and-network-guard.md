# web 工具中的不可信内容与网络防护

## 起点与终点

- 起点：模型调用 `web_search` 或 `web_fetch`
- 终点：外部网页内容被当作“数据”输入，而不是“指令”执行

## 不可信内容原则（防提示注入）

`web.py` 明确把外部网页视为不可信输入，并在内容侧增加防提示注入语义：

- 统一加入 `_UNTRUSTED_BANNER`
- 强制返回结构化 JSON，包括 `untrusted=true`
- 提醒模型“只把内容当数据，不当指令”

源码锚点：

- banner 常量：[web.py:L21-L25](file:///Users/bowhead/nanobot/nanobot/agent/tools/web.py#L21-L25)
- Jina 路径注入：[web.py:L265-L276](file:///Users/bowhead/nanobot/nanobot/agent/tools/web.py#L265-L276)
- readability 路径注入：[web.py:L312-L321](file:///Users/bowhead/nanobot/nanobot/agent/tools/web.py#L312-L321)

## SSRF 与网络边界

- `web_fetch` 在发请求前先做 URL 安全校验
- 仅允许 `http/https`，并校验 DNS 解析结果是否落入私网网段
- 跳转后会再次校验最终地址，防止“公开域名 -> 私网 IP”重定向
- 最大重定向次数固定为 5，降低重定向滥用风险

源码锚点：

- fetch 前校验：[web.py:L234-L239](file:///Users/bowhead/nanobot/nanobot/agent/tools/web.py#L234-L239)
- 重定向后校验：[web.py:L295-L299](file:///Users/bowhead/nanobot/nanobot/agent/tools/web.py#L295-L299)
- SSRF 核心规则：[network.py:L10-L63](file:///Users/bowhead/nanobot/nanobot/security/network.py#L10-L63)
- 重定向目标校验：[network.py:L65-L94](file:///Users/bowhead/nanobot/nanobot/security/network.py#L65-L94)
- 重定向上限常量：[web.py:L22-L24](file:///Users/bowhead/nanobot/nanobot/agent/tools/web.py#L22-L24)

## 关键测试覆盖

- SSRF 拦截与公网放行：[test_security_network.py:L41-L79](file:///Users/bowhead/nanobot/tests/test_security_network.py#L41-L79)
- `web_fetch` 私网与 localhost 拦截：[test_web_fetch_security.py:L22-L41](file:///Users/bowhead/nanobot/tests/test_web_fetch_security.py#L22-L41)
- `untrusted` 标记断言：[test_web_fetch_security.py:L44-L69](file:///Users/bowhead/nanobot/tests/test_web_fetch_security.py#L44-L69)

## 当前能力边界判断

- 强项：SSRF 防护和提示注入缓解已经形成默认路径
- 不足：不做域名级信誉判断，也无内建 rate limit
- 结论：适合作为“默认安全抓取层”，高风险场景需叠加网关策略
