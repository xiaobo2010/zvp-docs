# 用户（`/api/users`）

> 来源：`routers/users_router.py`　·　挂载前缀：`/api/users`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `POST` | `/api/users/auth` | `auth_user` | 公开 | body `AuthUser{id,mode,credential}` | 用户登录认证（发 token、密码强度、设备绑定观察） |
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
