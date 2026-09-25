# examples —— 报文示例

每个 HTTP 端点、每个 WebSocket 事件一组示例，**必须包含**：

- 成功请求 / 成功响应；
- 每一类错误的响应（至少：鉴权失败、角色不足、限流、冲突、体积超限）；
- 涉及幂等的端点要给出「重复请求返回同一结果」的第二组示例。

命名约定：`<域>/<端点或事件>-<场景>.json`，例如 `tasks/create-ok.json`、`images/upload-too-large.json`。

CI 用 `schema/index.json` 逐个校验本目录的全部文件。
