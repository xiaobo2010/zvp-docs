# API 参考

本平台后端为 FastAPI 应用，所有接口返回 JSON。

## 基础约定

| 项 | 值 |
|---|---|
| API 根路径 | `/api/` |
| 版本分层 | `/api/…`（基础层）、`/api/v2/…`、`/api/v3/…` |
| 内容类型 | `application/json`（附件上传为 `multipart/form-data`） |
| 鉴权方式 | OAuth2 Bearer Token |
| 对象标识 | MongoDB ObjectId 字符串（下称 OID） |

## 鉴权

认证使用 **OAuth2 Bearer Token**，请求头携带：

```http
Authorization: Bearer <token>
```

- 令牌通过登录端点 `POST /api/users/auth` 签发，请求体为 `{id, mode, credential, purpose?}`（**非** OAuth2 Password 表单）。代码虽以 `OAuth2PasswordBearer(tokenUrl="token")` 声明依赖，但平台并未暴露 `/token` 端点。
- 令牌含作用域区分：部分写操作要求**短期令牌**（`compulsory_temporary_token`）。
- 缺少或无效令牌时返回：

```json
{ "detail": "Could not validate credentials" }
```

状态码 `401`，并带 `WWW-Authenticate: Bearer` 响应头。部分端点允许匿名访问（`optional_current_user`），未登录时按访客裁剪数据。

## 权限

- 授权基于**权限组**（团队）判定，辅以角色。
- 无权限时返回 `403`：

```json
{ "detail": "权限不足" }
```

- 部分列表端点对非特权用户**静默降级**（返回裁剪结果），而非报错——调用方需以实际返回内容为准。

## 错误格式

统一为 `{"detail": <string>}`，随 HTTP 状态码返回：

| 状态码 | 含义 |
|---|---|
| `400` | 请求非法，例如 OID 格式错误：`{"detail": "Invalid Object ID"}` |
| `401` | 未认证 / 令牌无效 |
| `403` | 已认证但无权限 |
| `404` | 资源不存在 |
| `422` | 请求参数校验失败（FastAPI 默认结构） |

## 路由挂载

后端各路由模块的挂载前缀如下（取自 `main.py`）：

| 前缀 | 模块 | 分组 |
|---|---|---|
| `/api/users` | `users_router` | users |
| `/api/groups` | `groups_router` | groups |
| `/api/exports` | `exports_router` | exports |
| `/api/imports` | `imports_router` | imports |
| `/api/logs` | `logs_router` | logs |
| `/api/changelog` | `changelog_router` | changelog |
| `/api/attachments` | `attachments_router` | attachments |
| `/api/system/maintenance` | `system_router` | System |
| `/api/v2/activities` | `activities_v2_router` | — |
| `/api/v2/users` | `users_v2_router` | — |
| `/api/v2/groups` | `groups_v2_router` | — |
| `/api/v2/times` | `times_router` | — |
| `/api/v2/statistics/activities` | `activity_statistics_router` | — |
| `/api/v2/mcp` | `mcp_router` | — |
| `/api/v3/activities` | `activities_v3_router` | — |
| `/api/v3/users` | `users_v3_router` | — |
| `/api/v3/groups` | `groups_v3_router` | — |

## 维护模式

平台内建维护模式（`/api/system/maintenance`，详见 [系统](system.md)）。开启后：

- 业务 `/api/**` 请求一律 `503 {"detail": "系统维护中，请稍后再试", "maintenance": true}`；**admin 也无豁免**。
- 豁免：`/api`、`/api/`、`/api/users/auth`、`/api/cert`、`/api/version`、`/api/health`、整个 `/api/system` 前缀，以及所有 `OPTIONS` 预检。
- `/socket.io` 在维护期间握手即被拒（HTTP `503`，响应体含 `maintenance` 与 `socket`）；已建立的连接在开启瞬间被主动断开。

!!! warning "调试注意"
    平台有个「盾牌」中间件：`User-Agent` 不含 `Mozilla` 的请求一律返回 `418`。用 `curl` / 脚本调试时须带 `-A "Mozilla/5.0"`，否则会把 `418` 误判成业务失败。

## 逐端点参考

按业务域分页的端点清单（路径、方法、鉴权依赖、参数与用途）：

- 基础层：[用户](users.md)　·　[分组](groups.md)　·　[日志](logs.md)　·　[附件](attachments.md)　·　[系统](system.md)　·　[导入导出](transfers.md)　·　[更新日志](changelog.md)
- v2：[用户](users_v2.md)　·　[分组](groups_v2.md)　·　[活动](activities_v2.md)　·　[时长](times.md)　·　[统计](statistics.md)　·　[MCP](mcp.md)
- v3：[用户](users_v3.md)　·　[分组](groups_v3.md)　·　[活动](activities_v3.md)
