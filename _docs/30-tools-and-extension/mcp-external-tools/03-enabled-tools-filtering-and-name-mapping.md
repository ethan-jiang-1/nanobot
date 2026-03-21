# enabledTools 过滤与命名映射

## 起点与终点

- 起点：MCP server 返回一组远端工具定义
- 终点：仅符合 `enabled_tools` 规则的工具被注册到 ToolRegistry

## 双命名空间

过滤时支持两种命名：

- raw 名称：远端原始工具名（如 `read_file`）
- wrapped 名称：本地包装名（如 `mcp_filesystem_read_file`）

源码锚点：

- raw/wrapped 名称生成：[mcp.py:L145-L149](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L145-L149)
- 过滤条件：[mcp.py:L149-L159](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L149-L159)

## 默认与特殊取值

- `["*"]`：允许全部工具
- `[]`：不注册任何工具
- 其他列表：按显式名称子集注册

源码锚点：

- allow_all 判定：[mcp.py:L141-L143](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L141-L143)
- 配置定义注释：[schema.py:L134-L143](file:///Users/bowhead/nanobot/nanobot/config/schema.py#L134-L143)

## 未匹配项告警

当 `enabled_tools` 包含不存在名称时，会输出告警，并附带可选 raw/wrapped 名称，方便运维排查配置错误。

源码锚点：

- 未匹配告警：[mcp.py:L170-L180](file:///Users/bowhead/nanobot/nanobot/agent/tools/mcp.py#L170-L180)

## 相关测试

- raw/wrapped/default/empty/unknown 全覆盖：[test_mcp_tool.py:L174-L282](file:///Users/bowhead/nanobot/tests/test_mcp_tool.py#L174-L282)
