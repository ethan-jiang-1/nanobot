# 消息结构与多模态封装

## 目标

这篇聚焦 `_build_user_content` 如何把文本与媒体输入标准化成 provider 可接受的消息结构。

## 无媒体时的结构

当 `media` 为空，直接返回纯文本字符串，不引入额外块结构。

源码锚点：

- [context.py:L147-L151](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L147-L151)

## 有媒体时的处理流程

每个路径依次经过：

1. `Path(path).is_file()` 校验
2. `read_bytes()` 读取原始字节
3. `detect_image_mime(raw)` 按魔数识别真实 MIME
4. 若魔数失败，回退 `mimetypes.guess_type`
5. 非 image/* 一律跳过
6. base64 编码为 data URL
7. 生成 OpenAI 风格 `image_url` 块

源码锚点：

- [context.py:L152-L167](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L152-L167)
- MIME 识别实现：[helpers.py:L10-L23](file:///Users/bowhead/nanobot/nanobot/utils/helpers.py#L10-L23)

## 最终内容顺序

当至少有一张有效图片时，返回：

- 图片块数组
- 最后追加文本块 `{"type": "text", "text": text}`

源码锚点：

- [context.py:L169-L171](file:///Users/bowhead/nanobot/nanobot/agent/context.py#L169-L171)

## `_meta.path` 的作用

每个图片块会附带 `_meta.path`。这不是给模型看的语义字段，而是给后续持久化时做可读占位转换：

- `[image: /path/to/file]`
- `[image]`

相关实现与测试：

- 持久化占位：[loop.py:L487-L499](file:///Users/bowhead/nanobot/nanobot/agent/loop.py#L487-L499)
- 回归测试：[test_loop_save_turn.py:L25-L61](file:///Users/bowhead/nanobot/tests/test_loop_save_turn.py#L25-L61)

## 过滤策略的意义

- 非文件路径自动忽略
- 非图片 MIME 自动忽略
- 若全部媒体无效，则退回纯文本

这保证了“多模态可选增强”，而不会让坏输入破坏主链路可用性。
