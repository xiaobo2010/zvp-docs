# 分组 v3（`/api/v3/groups`）

> 来源：`routers/groups_v3_router.py`　·　挂载前缀：`/api/v3/groups`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/v3/groups/{group_id}/users` | `get_group_users_v3` | 登录态；门禁 _require_group_read_access (admin/department 或 secretary 本班) | query `page,size,query,pwdm` | 班级/权限组成员列表 |
| `POST` | `/api/v3/groups` | `create_group_v3` | 登录态、记日志；门禁 admin | body `NewGroup{name}` | 创建班级 |
| `PUT` | `/api/v3/groups/{group_id}/name` | `rename_group_v3` | 登录态、记日志；门禁 admin | body `RenameGroupRequest{name}` | 班级改名 |
| `DELETE` | `/api/v3/groups/{group_id}` | `delete_group_v3` | 登录态、记日志；门禁 admin | path `group_id` | 删除班级 |
| `POST` | `/api/v3/groups/{group_id}/users/{user_id}` | `add_group_user_v3` | 登录态、记日志；门禁 _is_group_manager (admin/department) | path `group_id,user_id` | 用户加入班级 |
| `DELETE` | `/api/v3/groups/{group_id}/users/{user_id}` | `remove_group_user_v3` | 登录态、记日志；门禁 _is_group_manager (admin/department) | path `group_id,user_id` | 用户移出班级 |
