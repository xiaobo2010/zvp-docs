# 日志（`/api/logs`）

> 来源：`routers/logs_router.py`　·　挂载前缀：`/api/logs`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/logs` | `read_logs` | 登录态、读预算；门禁 admin 或 `logs.read` | query `performer,page,perpage,query,type,level,time_start,time_end,xuehai_id` | 读取系统日志 |

日志端点**只读**（GET-only），没有写/删接口——审计记录不接受在线篡改。

## 请求参数

| 参数 | 说明 |
|---|---|
| `page` / `perpage` | 1 起；`perpage` 上限 100 |
| `query` | 在 `url` / `data` 上做大小写不敏感包含匹配；长度上限由运行期设置 `ops.max_query_count` 决定（超长 400） |
| `type` | 逗号分隔的 `ltype` 白名单 |
| `level` | 逗号分隔的级别（`info` / `warning` / `error` 等） |
| `performer` | 按操作者过滤 |
| `time_start` / `time_end` | epoch 秒；闭区间 |
| `xuehai_id` | 按学海设备 ID 等值过滤 |

- 不传 `type` 时默认隐藏 `login_fail`（两条风控规则的计数数据源，量大且对管理员无事可做）；显式 `type=login_fail` 仍可查回，账号排查入口保留。
- 返回 `data` 为行数组，`metadata.size` 为总数；行里另加 `user_name`（查不到时「未知用户」，未登录为「未登录」）。

## 行字段（本轮新增）

`APP_LOGS` 在 V045 补了两列，登录类事件与多数记录点都带：

| 列 | 类型 | 含义 |
|---|---|---|
| `client_ip` | `VARCHAR2(45)` | 真实来源 IP。生产在 nginx 之后、uvicorn 只听回环，取 `X-Forwarded-For` 的第一跳；取不到落空串（Oracle 空串即 NULL） |
| `user_agent` | `VARCHAR2(256)` | 客户端自述 UA，超长在写入侧截断 |

两列都可空、无默认值：历史行没有这个事实就是 `NULL`，不编占位值。当前**没有**按 IP 的索引——没有对应查询，索引只会拖慢写路径与保留期批删。

## 事件类型（`ltype`）

事件名统一收在 `services/log_events.py`（`VARCHAR2(64)`，无枚举约束）。本轮新增：

| 常量 | 值 | 场景 |
|---|---|---|
| `USER_LOGOUT` | `user_logout` | 登出成功（带 IP/UA） |
| `AI_QUOTA_REQUEST_APPROVE` | `ai_quota_request_approve` | 管理员批准 AI 配额申请 |
| `AI_QUOTA_REQUEST_REJECT` | `ai_quota_request_reject` | 管理员驳回 AI 配额申请 |

其余既有分组：登录/认证（`user_login` / `user_verify` / `login_fail` / `otp` / `passwd_change` / `auth_maintenance_blocked`）、用户与班级、封禁与设备、导入导出、附件、通知与更新日志、系统运维、AI 网关（`ai_*`）、平台误用上报。新增事件必须先加常量再引用字面量，前端 label 与常量同名对齐。

## 读预算

`GET /api/logs` 属于「重查询」，与 `GET /api/ai/logs`、`GET /api/ai/prompts`、`GET /api/exports/{id}/file` 共用每用户 **60 次/分钟**滑动窗口。超限：

```json
{ "detail": { "code": "READ_RATE_LIMITED", "message": "查询过于频繁，请稍后再试" } }
```

状态码 `429`，带 `Retry-After`。管理员与普通用户同一档位——预算挡的是「一个账号把库拖死」，与权限高低无关。

## 保留期

按 `APP_LOGS_RETENTION_DAYS`（默认 90 天）由定时任务批删；`warning` 级别的安全事件在保护集内，不随普通行清理。清理口径以 `ts_raw`（原始 epoch 秒）为准，`ts_at` 只作派生数据保留（V047）。
