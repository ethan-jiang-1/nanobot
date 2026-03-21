# filesystem 工具的路径约束与分页语义

## 起点与终点

- 起点：LLM 调用 `read_file / write_file / edit_file / list_dir`
- 终点：在路径边界内完成文件读写或目录浏览，并返回可继续迭代的结果

## 路径解析与边界

filesystem 工具共享 `_resolve_path`：

- 相对路径会基于 workspace 解析
- 开启 `allowed_dir` 时会校验目标是否位于允许目录下
- `read_file` 允许额外白名单目录（用于内置 skills 只读）

源码锚点：

- 路径解析：[filesystem.py:L9-L24](file:///Users/bowhead/nanobot/nanobot/agent/tools/filesystem.py#L9-L24)
- 目录包含判断：[filesystem.py:L27-L32](file:///Users/bowhead/nanobot/nanobot/agent/tools/filesystem.py#L27-L32)

## read_file 的分页协议

`read_file` 支持 `offset/limit`，并返回行号，便于模型分段读取大文件。  
当内容过大时会裁剪并提示下一段 offset，避免一次性注入过多上下文。

源码锚点：

- read_file 描述与参数：[filesystem.py:L53-L92](file:///Users/bowhead/nanobot/nanobot/agent/tools/filesystem.py#L53-L92)

## edit_file 与 list_dir 的模型友好性

- `edit_file` 支持轻微空白差异容错与 `replace_all`
- `list_dir` 支持递归与条目上限，超限时给出截断提示

源码锚点：

- edit_file 描述：[filesystem.py:L204-L219](file:///Users/bowhead/nanobot/nanobot/agent/tools/filesystem.py#L204-L219)
- list_dir 执行与截断：[filesystem.py:L339-L381](file:///Users/bowhead/nanobot/nanobot/agent/tools/filesystem.py#L339-L381)

## 相关测试

- 文件系统工具核心行为：[test_filesystem_tools.py:L17-L364](file:///Users/bowhead/nanobot/tests/test_filesystem_tools.py#L17-L364)
