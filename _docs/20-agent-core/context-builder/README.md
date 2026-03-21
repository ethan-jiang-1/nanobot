# context-builder

本目录用于拆解 ContextBuilder 如何把运行环境和历史对话拼成模型输入。

## 本主题要回答的问题

- 系统提示词由哪些部分组成
- 运行时元信息如何注入且避免污染指令
- 文本与多模态内容如何统一封装
- assistant/tool 消息追加时的结构约束是什么

## 建议文档拆分

- `01-system-prompt-composition.md`：身份、bootstrap、memory、skills
- `02-runtime-context-injection.md`：runtime tag 与 channel/chat 注入机制
- `03-message-shape-and-multimodal.md`：消息结构与图片内容编码

## 分析抓手

- `build_system_prompt`
- `build_messages`
- `_build_user_content`
