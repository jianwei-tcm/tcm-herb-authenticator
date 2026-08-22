# 真假药材分析系统 API 文档（MVP 正式版）

| 属性 | 值 |
|---|---|
| 文档版本 | 1.2 |
| 文档状态 | 正式版 |
| Base URL | `/api/v1` |
| 鉴权方式 | `Authorization: Bearer <access_token>` |
| 内容类型 | JSON；文件使用预签名 URL 上传 |
| 更新日期 | 2026-08-22 |

## 1. 范围与统一约定

本版本包含手机号/第三方登录、图片上传、异步药材鉴别、药材库、历史记录和投诉反馈。社区、养生堂、挑战赛及后台药材维护接口暂不纳入。

统一响应结构：

```json
{
  "code": "OK",
  "message": "success",
  "data": {},
  "request_id": "req_01J..."
}
```

分页请求使用 `page`（从 1 开始）和 `page_size`（默认 20，最大 100）；分页响应包含 `items`、`page`、`page_size`、`total`。时间统一使用 ISO 8601 UTC。创建鉴别任务必须携带 `Idempotency-Key`，客户端重试时复用同一值。

图片统一以图片对象返回：`{"asset_id":"10001","url":"https://storage.example/..."}`，`url` 为短时效签名 URL（有效期 30 分钟）。客户端直接将 `url` 作为图片地址使用，GET 请求无需携带 `Authorization`；不得缓存或持久化，签名过期（存储端返回 403）时重新请求对应接口刷新。

## 2. 错误码

| HTTP | code | 含义 |
|---:|---|---|
| 400 | `INVALID_ARGUMENT` | 参数格式或校验失败 |
| 401 | `UNAUTHORIZED` | 未登录或令牌无效 |
| 403 | `FORBIDDEN` | 无权访问资源 |
| 404 | `NOT_FOUND` | 资源不存在或不可见 |
| 409 | `CONFLICT` | 重复注册、重复操作或幂等冲突 |
| 409 | `RESULT_NOT_READY` | 鉴别尚未完成 |
| 413 | `FILE_TOO_LARGE` | 文件超过 10 MB |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | 文件类型不支持 |
| 422 | `BUSINESS_RULE_VIOLATION` | 不满足业务规则 |
| 429 | `RATE_LIMITED` | 请求过于频繁 |
| 500 | `INTERNAL_ERROR` | 服务内部错误 |
| 503 | `SERVICE_UNAVAILABLE` | 鉴别或依赖服务暂不可用 |

## 3. 认证接口

### 3.1 发送验证码

`POST /auth/sms/send`，无需登录。

请求体：

```json
{"phone":"13800138000","purpose":"register"}
```

`purpose` 可为 `register`、`login`、`reset_password`、`bind`。成功返回：

```json
{"expires_in":300,"retry_after":60}
```

响应不得返回验证码明文。单手机号 60 秒内最多发送一次。

### 3.2 手机号注册

`POST /auth/register`，无需登录。

请求体：

```json
{"phone":"13800138000","code":"123456","password":"S3cure-password","nickname":"药材用户"}
```

成功返回 HTTP 201，并返回 `user`、`access_token`、`refresh_token`、`expires_in`。手机号已注册返回 `409`。

### 3.3 登录

`POST /auth/login`，无需登录。

密码登录：`{"phone":"13800138000","password":"S3cure-password"}`。验证码登录：`{"phone":"13800138000","code":"123456"}`。成功返回用户信息和令牌。

### 3.4 第三方登录

`POST /auth/oauth/{provider}`，`provider` 为 `wechat` 或 `qq`。

请求体：`{"authorization_code":"...","device_id":"..."}`。服务端向第三方换取身份后登录或创建绑定身份；第三方用户 ID 不作为本地用户主键。

### 3.5 刷新和注销

- `POST /auth/refresh`：请求体 `{"refresh_token":"..."}`，返回新的访问令牌，刷新令牌轮换使用。
- `POST /auth/logout`：需要登录，请求体 `{"refresh_token":"..."}`，撤销当前会话。

## 4. 文件接口

### 4.1 创建上传凭证

`POST /files/presign`，需要登录。

请求体：

```json
{"filename":"sample.jpg","mime_type":"image/jpeg","size_bytes":2048000,"sha256":"e3b0c44298fc1c149afbf4c8996fb924...","purpose":"identification"}
```

`purpose` 为 `identification`、`avatar` 或 `feedback`；`sha256` 为客户端计算的文件 SHA-256 十六进制指纹。成功返回：

```json
{"asset_id":"10001","upload_url":"https://storage.example/...","expires_in":600}
```

上传约定：

- 客户端使用 `PUT` 方法向 `upload_url` 直传图片原始二进制，单次直传，不使用分片或 multipart 表单。
- 请求仅需携带 `Content-Type`，且必须与 presign 请求中的 `mime_type` 完全一致；禁止携带 `Authorization`，鉴权信息已包含在 URL 签名参数中。
- 存储端返回 HTTP 200 即视为上传成功，客户端记录 `asset_id` 用于创建业务资源；返回 403 表示签名过期（`expires_in` 秒内有效），重新调用本接口获取新地址。

服务端在创建业务资源时校验对象存在性、MIME、大小、图片可读性和 SHA-256（与 presign 声明的指纹比对），未完成上传的 `asset_id` 将被拒绝。

## 5. 鉴别接口

### 5.1 创建鉴别任务

`POST /identifications`，需要登录，必须携带 `Idempotency-Key`。

请求体：

```json
{
  "asset_ids":["10001","10002"],
  "description":"颜色偏黄，气味较淡",
  "features":{
    "texture":"有明显纹理",
    "size":"约3厘米",
    "odor":"淡",
    "touch":"偏硬",
    "humidity":"干燥",
    "taste":"未提供"
  }
}
```

规则：`asset_ids` 必填，数量 1 至 5；文件必须属于当前用户且上传完成；描述最多 2000 字；特征字段均可选。成功返回 HTTP 202：

```json
{"id":"20001","status":"queued","submitted_at":"2026-08-21T12:00:00.000Z"}
```

### 5.2 查询鉴别任务

`GET /identifications/{id}`，需要登录且只能访问本人任务。

返回任务状态、`images`（图片对象数组，顺序与提交时一致）、描述、特征和失败信息。非终态不返回鉴别结论。

任务 `status` 为 `failed` 时返回 `failure_code`（机器可读的失败原因编码）和 `failure_message`（可直接展示的通用文案）；其余状态不返回这两个字段。`failure_code` 描述异步任务的终态失败原因，独立于第 2 节的 API 错误码，取值：

| 值 | 含义 | 建议提示方向 |
|---|---|---|
| `image_unreadable` | 图片无法解码或无法用于鉴别 | 引导用户重新拍摄清晰图片后再次提交 |
| `model_service_unavailable` | 鉴别服务暂不可用 | 稍后重试 |
| `model_timeout` | 分析超时 | 稍后重试 |
| `model_internal_error` | 模型内部错误 | 通用失败提示 |
| `invalid_model_response` | 鉴别结果校验未通过 | 通用失败提示 |
| `internal_error` | 服务内部错误，兜底 | 通用失败提示 |

客户端必须对未知 `failure_code` 做兜底展示（如“鉴别失败，请稍后重试”），以保证服务端后续新增枚举时旧版本客户端不受影响。`uncertain`、`unrecognized` 走结论字段，`cancelled` 为独立终态，均不携带 `failure_code`。

### 5.3 查询鉴别结果

`GET /identifications/{id}/result`，需要登录且只能访问本人任务。

任务处于 `succeeded` 或 `uncertain` 时返回：

```json
{
  "request_id":"20001",
  "status":"succeeded",
  "conclusion":"authentic",
  "material":{"id":"30001","name":"黄芪","category":"plant"},
  "recognition_confidence":0.98700,
  "authenticity_confidence":0.93100,
  "model":{"name":"herb-authenticity","version":"2026.08.1"},
  "explanation":"图像特征与参考样本较为一致。",
  "risk_notice":"本结果仅供信息参考，不能替代专业检验或医疗建议。",
  "reference_standards":[]
}
```

`conclusion` 可为 `authentic`、`counterfeit`、`uncertain`、`unrecognized`。后两者必须展示明确的不确定提示，不能包装成确定结论。任务未完成时返回 `409 RESULT_NOT_READY`。本接口不返回图片，原图通过任务详情接口（5.2）获取。

`material` 仅包含 `id`、`name`、`category` 定位信息。结果页需要展示的药材功效、注意事项、储存条件等详情由客户端另行调用药材详情接口（7.2 `GET /materials/{id}`）获取；`conclusion` 为 `unrecognized` 时可能没有 `material.id`，客户端需判断存在后再请求。

### 5.4 取消鉴别任务

`POST /identifications/{id}/cancel`，需要登录且只能访问本人任务。仅 `created`、`queued` 状态允许取消；成功返回任务状态 `cancelled`。

## 6. 历史记录接口

### 6.1 查询我的鉴别记录

`GET /me/identifications?page=1&page_size=20&status=succeeded&keyword=黄芪`，需要登录。

只返回当前用户任务，按 `submitted_at` 倒序。列表项包含 `id`、`status`、`first_image`（首张图片对象）、药材名称、结论、提交时间和完成时间。

### 6.2 查看我的记录详情

`GET /me/identifications/{id}`，需要登录。

返回任务、`images`（全部图片对象，顺序与提交时一致）、特征、结果、风险提示和参考标准。权限规则与任务详情接口相同。

## 7. 药材库接口

### 7.1 药材列表

`GET /materials?page=1&page_size=20&category=plant&keyword=黄芪`，无需登录。

`category` 为 `plant`、`animal`、`mineral`。仅返回 `published` 状态药材，支持名称和别名模糊查询。

### 7.2 药材详情

`GET /materials/{id}`，无需登录。

返回名称、别名、类别、学名、封面图、功效、注意事项、储存条件、内容来源和版本。

## 8. 反馈接口

### 8.1 提交反馈

`POST /feedback`，需要登录。

请求体：

```json
{"type":"identification","content":"结果与人工判断不一致","request_id":"20001","asset_ids":["10003"]}
```

`type` 为 `suggestion`、`complaint`、`identification`、`other`；内容 1 至 2000 字；图片最多 3 张且必须属于当前用户。成功返回 HTTP 201 和反馈 ID。

## 9. 模型服务内部回调

`POST /internal/identifications/{id}/callback`

仅允许机器学习服务通过服务身份调用，客户端禁止访问。请求至少包含 `status`、`model_name`、`model_version`、`conclusion`、置信度、解释和错误信息。服务端必须校验签名、任务 ID、状态转移合法性，并保证重复回调幂等。

## 10. 安全与合规

- 所有业务查询按当前登录用户做资源级鉴权。
- 私有图片只返回短时效签名 URL，原始模型输出不得返回客户端。
- 登录、验证码、鉴别创建和文件上传必须限流并记录审计日志。
- 结果页统一展示“仅供信息参考，不能替代专业检验或医疗建议”。
- API 不返回手机号、第三方凭证等敏感信息，日志必须脱敏。

## 11. 联调验收清单

- 注册、登录、刷新令牌、注销流程可正常完成。
- 图片格式、大小、数量和归属校验一致。
- 鉴别任务按 `created -> queued -> processing -> 终态` 流转。
- 前端能展示排队、分析中、成功、不确定、失败和取消状态。
- `uncertain`、`unrecognized` 不被显示为确定真伪。
- 用户无法读取其他用户的任务、图片和反馈。
- 模型回调重复提交不会生成重复结果。
- 图片上传遵循 PUT 直传约定，图片展示使用签名 `url` 字段，过期后重试接口可刷新。
