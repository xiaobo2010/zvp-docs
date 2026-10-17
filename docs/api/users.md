# 用户（`/api/users`）

> 来源：`routers/users_router.py`　·　挂载前缀：`/api/users`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `POST` | `/api/users/auth` | `auth_user` | 公开 | body `AuthUser{id,mode,credential,purpose?}` | 登录 / 身份验证（`purpose` 区分，见下） |
| `POST` | `/api/users` | `create_user` | 短期令牌 | body `User` | **（停用）** 返回 503，提示改用 `POST /api/v3/users` |
| `DELETE` | `/api/users/{target}` | `delete_user` | 无 | path `target` | **（停用）** 返回 503 用户删除已停用 |
| `PUT` | `/api/users/{user_oid}/password` | `change_password` | 短期令牌、记日志；门禁 admin/department/auditor/本人临时token/secretary同班 | body `PutPassword{credential}` | 修改/重置用户密码 |
| `GET` | `/api/users` | `read_users` | 登录态 | query `query,page,perpage,privilege` | 查询用户列表（分页/搜索） |
| `GET` | `/api/users/{user_oid}` | `read_user` | 登录态 | path `user_oid` | 读取用户信息；调用 validate_read_user_permission("volunteer") |
| `PUT` | `/api/users/{user_oid}` | `update_user` | 无 | body `PutUser` | **（停用）** 返回 503 用户信息修改已停用 |
| `POST` | `/api/users/{user_oid}/group` | `add_user_to_group` | 无 | query `group_id` | **（停用）** 返回 503，提示改用 `POST /api/v3/groups/{group_id}/users/{user_id}` |
| `DELETE` | `/api/users/{user_oid}/group/{group_id}` | `remove_user_from_group` | 无 | path `user_oid,group_id` | **（停用）** 返回 503，提示改用 `DELETE /api/v3/groups/{group_id}/users/{user_id}` |
| `GET` | `/api/users/{user_oid}/activities` | `read_user_activity` | 登录态；门禁 admin/department/本人 | query `page,perpage,query` | 读取用户的活动 |
| `GET` (api_route) | `/api/users/{user_oid}/activity/{path:path}` | `redirect_activity` | 无 | path | 重定向到 `/api/users/{user_oid}/activities/{path}` |
| `GET` | `/api/users/{user_oid}/time` | `read_user_time` | 登录态；门禁 admin/department/本人 | query `start,end` | 读取用户义工时长 |
| `GET` | `/api/users/{user_oid}/logs` | `read_logs` | 登录态；门禁 admin | query `page,perpage,query` | 读取用户日志 |
| `DELETE` | `/api/users/{user_oid}/past/{past_identity_idx}` | `delete_past` | 登录态、记日志；门禁 admin | path `past_identity_idx` | 删除用户历史身份 |
| `POST` | `/api/users/{user_oid}/past` | `add_past` | 登录态、记日志；门禁 admin | body `PostPast{past}` | 添加用户历史身份 |

## 认证（`POST /api/users/auth`）

请求体：

```json
{ "id": "1234567", "mode": "long", "credential": "<RSA 加密凭据>", "purpose": "login" }
```

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | string | 学号或用户 OID |
| `mode` | string | `long`（默认）或 `short`；空值按 `long` 处理 |
| `credential` | string | RSA 加密的 JSON（含 `time` 与 `password` 字段）；密码或动态码（TOTP） |
| `purpose` | string | `login`（默认）或 `verify`；**可选** |

### `purpose` 语义

- `login`：登录语义，可能签发**长期会话令牌**（`mode=long` 且密码强度通过时）。
- `verify`：身份验证语义（重置密码前验证、删除义工前认证等二次认证）。服务端**钳制**——**绝不下发会话令牌**：即使传 `mode=long`，也按 short 分支签发 **5 分钟临时令牌**（`scope=temporary_token`）。动态码（TOTP）在 `verify` 下同样可用，结果仍是临时令牌。
- **fail-safe**：只有严格等于 `"login"` 才走登录语义；其它任何值（拼错、空串、未知）**一律按 `verify` 处理**——拼错不会换来长期会话令牌。

### `mode` 语义

- `long`：长期会话令牌（`scope=access_token`，有效期 10 天）；密码强度不足时服务端**降级**为 5 分钟临时令牌，并置 `weak_password=true`。
- `short`：5 分钟临时令牌（`scope=temporary_token`），用于危险操作二次授权。
- 动态码（TOTP）以 `*<code>*` 形式提交；`mode=long` 时签发 1 天令牌。

### 响应与失败

成功返回：

```json
{ "token": "<jwt>", "_id": "<oid>", "profile": { }, "weak_password": false, "binding_mismatch": null }
```

- 「用户不存在」与「凭据错误」**不可区分**，同为 `403 {"detail": "用户名或密码错误"}`。
- 动态码错误或过期：`403 {"detail": "动态码错误或已过期"}`。
- 用户被封禁：`403 {"detail": "User is banned"}`。
- 凭据时间戳过期：`401 {"detail": "Token expired"}`。
- **审计**：成功时记 `user_login`（登录）或 `user_verify`（验证）两类日志；**失败不写日志**。
