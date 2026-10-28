# ZVAgent / AI 网关（`/api/ai`）

> 来源：`routers/ai_router.py`、`routers/ai_conversations_router.py`、`routers/ai_scenes_router.py`、`routers/ai_agent_router.py` 等　·　挂载前缀：`/api/ai`

AI 产品统一称 **ZVAgent**；「AI 网关」指本模块。它提供 OCR、OpenAI 兼容面、持久会话与后台 run、只读场景，以及凭据 / 引擎 / 额度 / 流水等运维面。完整契约（字段级、计费口径、压缩算法）见开发仓库 `docs/AI_GATEWAY_API.md` 与 `docs/AI_CONVERSATION_DESIGN.md`，本页只列 HTTP 事实。

## 鉴权与门禁

| 凭据 | 形态 | 作用面 |
|---|---|---|
| 平台登录 token | `Authorization: Bearer <jwt>` | 全部 `/api/ai/*` 与 `/api/ai/v1/*` |
| 独立 API key | `Authorization: Bearer zvk_...` | **仅** `/v1/models`、`/v1/chat/completions` |

角色门禁 `AI_ALLOWED_ROLES`（默认 `admin`、`department`）；管理面（凭据、引擎、额度、流水、健康、API key 管理）只认 `admin`。个人凭据（`scope=personal`）的调用不占平台月额度。

错误体分两套：`/api/ai/*` 用 `{"detail": "<错误码>"}`；OpenAI 兼容面用 `{"error":{"message","type","code"}}`。对话与场景请求还带 `X-AI-Request-Id`。

## 端点

| 方法 | 路径 | 权限 | 说明 |
|---|---|---|---|
| `POST` | `/api/ai/ocr` | AI 可用角色 | 附件 OCR 入队（202 新受理 / 200 已在处理） |
| `POST` | `/api/ai/ocr/file` | `admin` + `ai.ocr` | 站点文件路径同步 OCR，返回 `{text}` |
| `GET` | `/api/ai/keys` / `POST` / `PATCH` / `DELETE` | `admin`（写操作 **+ 再认证**） | 上游凭据管理 |
| `GET` | `/api/ai/ocr-engines` 等 | `admin` | OCR 引擎注册表 |
| `GET` | `/api/ai/quota` | 登录 | 本人 OCR / chat 月额度 |
| `GET` | `/api/ai/logs`、`/logs/stats`、`/prompts`、`/prompts/{oid}` | `admin` | 调用流水、聚合、受控 prompt 留存 |
| `GET` | `/api/ai/health` | `admin` | 网关健康与进程内阈值 |
| `GET` | `/api/ai/v1/models` | AI 可用角色 / `zvk_` | 可见模型（每项带 `context`） |
| `POST` | `/api/ai/v1/chat/completions` | AI 可用角色 / `zvk_` | **无状态** OpenAI 兼容对话 |
| `GET/POST/PATCH/DELETE` | `/api/ai/api-keys...` | 本人 / `admin` | 独立 API key（写操作 **+ 再认证**） |
| `POST` | `/api/ai/conversations` | 本人 | 建会话（`mode` 只剩 `agent`） |
| `GET` | `/api/ai/conversations` | 本人 | 会话列表（`status=active` / `archived`） |
| `PATCH` | `/api/ai/conversations/{oid}` | 本人 | 改名 / 归档恢复 / 思考强度；改模型 409 |
| `DELETE` | `/api/ai/conversations/{oid}` | 本人 | 硬删（有活跃 run 则 409） |
| `GET` | `/api/ai/conversations/{oid}/messages` | 本人 | `after_seq` 增量消息 |
| `POST` | `/api/ai/conversations/{oid}/runs` | 本人 + 解锁 | 提交后台 run（202） |
| `GET` | `/api/ai/runs/{run_id}` | 本人 | run 状态 + 事件 + 提案（可带增量消息） |
| `POST` | `/api/ai/runs/{run_id}/cancel` | 本人 | 请求中止 |
| `POST` | `/api/ai/scenes/{scene}/runs` | `admin` + `ai.agent` | 只读场景（`terminal` / `file_edit`） |

## 文件 OCR：`POST /api/ai/ocr/file`

```json
{ "path": "/www/zvms4/uploads/order.png", "provider": "tencent" }
```

- `path` 必须是绝对路径，经 `services/file_ops.resolve_path` 解析；**只允许 `/www/zvms4` 与 `/home`**，越界 403 `FILES_ROOT_FORBIDDEN`。
- 扩展名只认 `jpg` / `jpeg` / `png` / `webp`，否则 400 `AI_UNSUPPORTED_MEDIA`；体积上限默认 20MB，超限 400 `AI_OCR_IMAGE_TOO_LARGE`。
- 同步识别，成功 `200 {"status":"ok","code":200,"data":{"text":"..."}}`；不落 `file_ocr`、不产生附件配额。
- 计费与附件 OCR 同源：频控（`ocr`）+ 用户 `ocr` 月额度 + 站点额度 + 每次尝试的 `ai_call_logs`；成功另写一条 `APP_LOGS`（`type=ai_ocr_enqueue`）。
- `provider` 是引擎 kind（`tencent` / `baidu` / `rapidocr` / ...），仅 admin；拼错 400 `AI_OCR_ENGINE_BAD_KIND`。

## 会话与模型

创建会话：

```json
{ "mode": "agent", "model": "team/glm-4.6v-flash", "thinking": "medium", "title": "可选" }
```

| 字段 | 说明 |
|---|---|
| `mode` | 只剩 `agent`（缺省）。**`chat` 模式已下线**：新建回 409 `AI_CONVERSATION_MODE_MISMATCH`，存量 chat 会话只可读 / 归档 / 删除 |
| `model` | **建会话时锁定**；缺省取该用户可见目录的第一个。之后 `PATCH` 传 `model`（含 `null`）一律 409 `AI_CONVERSATION_MODEL_LOCKED` |
| `thinking` | `low` / `medium` / `high` / `null`（NULL = 不给上游附带字段）。上游翻译：`openai_compat → reasoning_effort`，智谱 → `thinking`（`low → disabled`） |
| `title` | 可选；缺省由首条用户消息截断生成 |

`PATCH` 的 `status` 在 `active` / `archived` 间切换即归档与恢复。归档不是删除：消息、run、提案都在。

提交 run 的 body 是 `{content, client_msg_id?}`——**不收 `model`**（执行模型一律取会话行；老客户端多传会被忽略而不是 422）。响应 `202 {"status":"ok","code":202,"data":{run_id,message_oid,status:"queued"}}`，之后用 run 详情轮询。

`GET /api/ai/runs/{run_id}`：

| 参数 | 说明 |
|---|---|
| `after_round` | 只回 `round_no > after_round` 的事件，轮询载荷线性增长 |
| `since_msg_seq` | 给了就额外带 `messages`（`seq > since_msg_seq`、升序、与消息端点同形状）；**不传则响应没有 `messages` 键** |

建议客户端持有 `last_seq`，每轮把上一轮最后一条的 `seq` 传回；`events` 与 `messages` 同响应到达，避免「run 结束、消息还没拉」的中间态。

## 只读场景：`POST /api/ai/scenes/{scene}/runs`

```json
{ "content": "这段报错是什么意思？", "context": "Traceback ...（终端输出或文件片段，≤8000）" }
```

| 项 | 规则 |
|---|---|
| `scene` | `terminal` 或 `file_edit`；拼错 400 `AI_SCENE_UNKNOWN` |
| body 上限 | `content` 1–4000；`context` ≤8000 |
| 权限 | 严格 `admin` + `ai.agent`；**跳过 30 分钟密码门**（只读 + admin） |
| 工具 | 只声明只读集（`kind='read'`）；写工具、提案流、`ssh_exec`、DML/DDL 都不存在 |
| 上下文 | `context` 先脱敏再截断，只进 prompt 与 prompt 留存，**不落消息表** |
| 会话 | 按 `(user, scene)` 复用；标题固定「终端智能」/「文件编辑智能」；不出现在普通会话列表（`scene IS NULL` 过滤） |
| 响应 | `202 {"status":"ok","code":202,"data":{"run_id","conversation_oid"}}`，用上面的 run 详情轮询 |
| 保留 | 场景会话 14 天（级联删除 messages / runs） |

错误体与 `/api/ai/conversations` 同形（`{"error":{message,type,code}}` + `X-AI-Request-Id`）。

## 内容安全

提交路径先过关键词规则、再过一个严格判定的 LLM 灰区闸，**fail-closed**：判定跑不起来（无凭据 / 超时 / 5xx / 坏 JSON）也拒绝本次回答，但不记违规。判定成立时对外只有：

```json
{ "error": { "message": "该请求未通过内容安全审核。", "code": "AI_SAFETY_REJECTED" } }
```

- run 提交：HTTP `422`；`/v1/chat/completions`：HTTP `403`。
- 对内记违规 + 警告 + 按 `BAN_RULES.penalties` **首档立即封号**；同一份证据 5 分钟内去重。
- `admin` 天然旁路（每次旁路写 `ai_moderation_unrestricted` 审计）；「管理员无限制模式」开关已下线。
- 只对 `department` 生效：它挡的是灰区破坏性请求，不是全站门禁。

## OpenAI 兼容面

`GET /api/ai/v1/models` 只列当前用户可见凭据声明的模型，`id` 是合格 ID `provider/model`，每项带 `context`（上下文窗口 token，未知为 `null`）。凭据的 `MODELS_JSON` 支持字符串数组与 `[{"id","context"}]` 两种写法；保存时服务端 best-effort 探测上游 `/models` 补空缺，管理员手填优先。

`POST /api/ai/v1/chat/completions` 按 OpenAI 规范透传，网关只读 `model`、`messages`、`stream`、`provider`；`provider` 仅 admin。它是**无状态**的，不读写会话——需要持久历史请用上面的会话端点。

## 错误码

| code | HTTP | 含义 |
|---|---|---|
| `AI_FORBIDDEN` | 403 | 角色无 AI 权限 / 非 admin 用管理面 |
| `AI_SAFETY_REJECTED` | 422 / 403 | 内容安全拒绝（run / `/v1`），固定文案 |
| `FILES_ROOT_FORBIDDEN` | 403 | OCR 路径越出 `/www/zvms4`、`/home` |
| `AI_UNSUPPORTED_MEDIA` | 400 | OCR 类型不支持 |
| `AI_OCR_IMAGE_TOO_LARGE` | 400 | 单图超上限 |
| `AI_RATE_LIMITED` | 429 | 本地频控（带 `Retry-After`） |
| `AI_QUOTA_EXCEEDED` / `insufficient_quota` | 429 | 月额度耗尽 |
| `READ_RATE_LIMITED` | 429 | 重查询读预算（日志 / prompt / 导出下载） |
| `AI_CONVERSATION_MODEL_LOCKED` | 409 | 试图改已锁定的模型 |
| `AI_CONVERSATION_MODE_MISMATCH` | 409 | 会话模式不符（`chat` 已下线） |
| `AI_SCENE_UNKNOWN` | 400 | 场景名不是 `terminal` / `file_edit` |
| `AI_RUN_CONFLICT` / `AI_RUN_NOT_FOUND` / `AI_RUN_NOT_CANCELLABLE` | 409 / 404 / 409 | run 冲突 / 不存在 / 已终态 |
| `AI_AGENT_UNLOCK_REQUIRED` | 422 | 普通 agent 未通过 30 分钟密码解锁 |
| `AI_TIMEOUT` / `AI_UPSTREAM_*` / `AI_CREDENTIAL_UNREADABLE` | 504 / 502 | 上游超时 / 失败 / 凭据不可读 |

## 再认证与读预算

- `/api/ai/keys`、`/api/ai/api-keys` 的写操作要求 body `verification_token`（5 分钟密码短令牌）；缺失或无效 → 403 `需要密码验证后才能继续`。
- `GET /api/ai/logs`、`GET /api/ai/prompts` 与 `GET /api/logs`、`GET /api/exports/{id}/file` 共用每用户 60 次/分钟读预算；超限 429 `READ_RATE_LIMITED`。
