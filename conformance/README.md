# conformance —— 一致性向量（三端实现的唯一裁判）

向量是**语言无关的数据**：给定初始状态与一串输入（HTTP 请求 / WS 消息 / 时钟推进），规定每一步的期望可观测结果。任何一端（Dart 参考实现、Kotlin、C#）都回放同一批向量。

> 纪律：**向量失败就是实现错**。确实要改向量时，必须同一次提交里改规范并写明理由（见根 README 的评审红线）。

## 1. 文件组织

| 路径 | 内容 | 状态 |
|---|---|---|
| `vectors/auth.ndjson` | 鉴权：令牌缺失/错误/已吊销、配对码、限流、版本协商 | **23 步，已通过** |
| `vectors/images.ndjson` | 上传与下载：去重、体积两阶段拒绝、频率限制、hash 白名单 | **14 步，已通过** |
| `vectors/tasks.ndjson` | 任务：幂等创建、结果复用、页数上限、队列深度、失败语义 | **18 步，已通过** |
| `vectors/sync.ndjson` | 同步：Lamport、逐字段 LWW、op 归属、墓碑、水位游标、快照分页 | **32 步，已通过** |
| `vectors/errors.ndjson` | 错误码与协商的边角：malformed 请求、空/畸形鉴权头、路由缺失、响应里不得出现令牌哈希 | **14 步，已通过** |
| `vectors/websocket.ndjson` | WS：hello、事件、心跳、重连、踢旧 | **待写**（回放器的 `ws` 操作尚未实现） |

一个文件 = 一个场景序列，**步骤在同一台服务端上按顺序执行**；回放器为**每个文件**起一套干净的服务端 + 客户端。

## 2. 行格式（NDJSON：一行一个 JSON 对象）

```jsonc
{"step": 5, "title": "正确的配对码 → 200 并签发 token",
 "do": {"http": {"method": "POST", "path": "/api/v1/pair", "auth": "none",
                 "json": {"code": "$pairingCode", "device_id": "android-device-1",
                          "device_name": "Pixel 7", "platform": "android", "app_version": "1.0.0"}}},
 "expect": {"status": 200,
            "json": {"server_device_id": "server-device-1", "server_name": "TEST-HOST", "protocol_version": 1},
            "json_regex": {"token": "^[0-9a-f]{64}$"},
            "capture": {"token": "$.token"}}}
```

- `step`：1 起、必须连续（回放器校验，跳号即失败）。
- `title`：给人看的一句话。
- `do`：**恰好一个**操作（见 §3）。
- `expect`：断言（见 §4）；纯动作步骤可以省略。

## 3. 操作（`do`）

| 形态 | 含义 | 状态 |
|---|---|---|
| `{"http": {"method": "GET\|POST\|DELETE", "path": "/api/v1/…", "auth": …, "json": {…} \| "raw": "<原样字节>" \| "multipart": {…}, "headers": {…}, "repeat": N}}` | 发一次真实 HTTP 请求。`json` = JSON body；`raw` = 原样字符串 body（测「body 不是合法 JSON」）；`multipart` = `{"field":"file","bytes":<长度>,"filename":"x.jpg"}`（回放器合成指定长度的字节，用来测体积上限）；`headers` 用于版本协商一类用例；`repeat: N` = 同一请求连发 N 次（每次按 `expect` 断言），用来表达「同一个动作来很多次」的限流用例 | 已实现 |
| `auth` 取值 | `none`（不带 Authorization）/ `device`（默认 token）/ `device:<名字>`（capture 出来的命名 token）/ `bogus`（64 个 0）/ `empty`（`Bearer ` 空令牌）/ `malformed`（没有 Bearer 前缀） | 已实现 |
| `{"clock": {"advance_ms": 300001}}` | 推进**回放器注入的时钟**（服务端通过 `now` 注入拿到它），用于配对码过期 / 限流窗口 / 锁定期 | 已实现 |
| `{"server": {"refresh_pairing_code": true}}` | 调服务端刷新配对码（等价 UI 上的「刷新」，会重置有效期） | 已实现 |
| `{"seed": {"collection": {"id": "c1", "name": "期末复习"}, "active_collection": "c1", "queued_tasks": 20}}` | 回放器侧的初始状态：主机库里的合集、当前选中合集、已排队任务数（用来撞队列深度上限） | 已实现 |
| `{"poll": {"method": "GET", "path": "/api/v1/tasks/t-1", "auth": "device", "until": {"status": "done"}, "max_ms": 20000, "interval_ms": 100}}` | 轮询只读端点直到 `until` 的局部匹配成立（或超时失败），随后按外层 `expect` 断言最后一次响应 | 已实现 |
| `{"ws": {"connect": "device"}}`、`{"ws": {"expect": {…}}}`、`{"ws": {"send": {…}}}`、`{"ws": {"close": true}}` | WebSocket 连接 / 等一条事件 / 发一条消息 / 关闭 | **待实现** |

> **未实现的操作 = 回放失败**（不是跳过）：宁可红着，也不要假的绿。

## 4. 断言（`expect`）

| 字段 | 含义 |
|---|---|
| `status` | HTTP 状态码（精确相等） |
| `json` | **深度局部匹配（数组严格）**：对象只要求期望的键存在且递归匹配；**数组要求长度相等、按下标逐项递归匹配**（所以 `[]` 断言「必须是空数组」）；标量精确相等。期望值里可以写 `$变量` |
| `json_contains` | 同上，但**数组只要求包含**期望里的每一项（顺序无关）。用于「顺序不是契约」的清单（如 `/info` 的 `capabilities`）与「数组里存在这样一条记录」的断言 |
| `json_regex` | `{JSON 路径: 正则}`，例如 `{"token": "^[0-9a-f]{64}$"}` |
| `capture` | `{变量名: JSON 路径}`，把响应里的值存成变量，供后续步骤以 `$变量名` 引用 |
| `json_absent` | `[JSON 路径, …]`：断言这些路径**不存在**（例：设备列表里绝不能出现令牌哈希） |
| `body_sha256` | 响应体字节的 sha256（用来证明「下载到的就是上传的那张图」：与 capture 出的 `image_hash` 相等） |

JSON 路径支持 `a.b`、`a[0].b` 两种写法。

> 目前**没有**响应头断言 → `X-QS-Server-Version` 这类响应头尚无向量覆盖（缺口，见 §1 的状态表）。

## 5. 变量

| 变量 | 来源 |
|---|---|
| `$pairingCode` | 回放器**每次使用时实时读取**服务端当前配对码（刷新后自动跟着变） |
| `$port` | 回放器监听的端口 |
| 其它 | 由 `capture` 产生（例如 `$token`、`$token2`、`$hash`、`$session_id`） |

## 6. 回放约定

- **回放器只做 I/O 适配**：直接用 `HttpClient` / `WebSocket` 打真实的 HTTP / WS，**不引用任何一端的客户端实现**（拿被裁判的实现去断言，就不是裁判了），也不在里面写业务逻辑。
- Dart 参考回放器在 `QuizSyncAI` 仓库：`packages/quizsync_core/test/conformance/vector_replay_test.dart`，向量目录用环境变量 `QS_VECTORS` 指定（默认 `../../../QuizSyncProtocol/conformance/vectors`）。**目录不存在 = 失败**（不允许静默跳过）。
- 参考回放器必须 **100% 通过**：v1 快照描述的就是现有实现的行为，红一条就说明快照或向量写错了。
