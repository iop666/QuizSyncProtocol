# schema —— JSON Schema 2020-12

组织方式：`envelope.json`（WebSocket 信封与 HTTP 错误体）+ 每种消息一个 schema；实体 schema 被消息 `$ref` 引用。

| 路径 | 内容 | 状态 |
|---|---|---|
| `index.json` | 全部 schema 的索引（CI 按它逐个校验 examples） | 待写 |
| `envelope.json` | WS 信封 `{v, type, id, ts, corr, payload}`、HTTP 错误体 | 待写 |
| `entities/` | 会话、题目、选项、合集、设备、op、任务… | 待写 |
| `messages/` | 每个 HTTP 端点与每个 WS 事件的请求/响应 | 待写 |

## 纪律

- **v2.0 不引入代码生成流水线**：手写模型 + 向量回放保证一致，避免提前引入构建复杂度。
- Schema 是**校验用**的，不是规范正文：语义写在 `spec/`，Schema 只保证结构。两者冲突时以 `spec/` 为准并立刻修 Schema。
- 每个 schema 必须给出 `additionalProperties` 的明确选择（默认允许未知字段，与「未知字段一律忽略」的协商规则一致）。
