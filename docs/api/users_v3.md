# 用户 v3（`/api/v3/users`）

> 来源：`routers/users_v3_router.py`　·　挂载前缀：`/api/v3/users`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/v3/users/{user_id}` | `get_user_v3` | 登录态；门禁 本人/admin/department/secretary同班 | path `user_id` | 读取用户资料 |
| `PUT` | `/api/v3/users/{user_id}/roles` | `modify_user_roles_v3` | 登录态、记日志；门禁 admin | body `PutRolesRequest{roles}` | 修改用户角色 |
| `PUT` | `/api/v3/users/{user_id}/banned` | `set_user_banned_v3` | 登录态、记日志；门禁 admin | body `PutBannedRequest{banned}` | 封禁/解封用户 |
| `POST` | `/api/v3/users` | `create_user_v3` | 登录态、记日志；门禁 admin | body `NewUserRequest{id,name,roles,class_id}` | 创建用户 |
| `DELETE` | `/api/v3/users/{user_id}/password` | `reset_user_password_v3` | 短期令牌、记日志；门禁 admin/department/secretary同班 | path `user_id` | 重置密码为学号 |
| `GET` | `/api/v3/users/{user_id}/totp` | `get_user_totp_v3` | 登录态、记日志；门禁 admin | path `user_id` | 查看用户动态码 |
| `GET` | `/api/v3/users/{user_id}/bindings` | `list_user_bindings_v3` | 登录态；门禁 admin | path `user_id` | 列出学海设备绑定 |
| `PUT` | `/api/v3/users/{user_id}/bindings` | `set_user_binding_v3` | 登录态、记日志；门禁 admin | body `PutBindingRequest{xuehai_id,note}` | 绑定学海设备 |
| `DELETE` | `/api/v3/users/{user_id}/bindings/{xuehai_id}` | `delete_user_binding_v3` | 登录态、记日志；门禁 admin | path `xuehai_id` | 解绑学海设备 |
