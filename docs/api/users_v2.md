# 用户 v2（`/api/v2/users`）

> 来源：`routers/users_v2_router.py`　·　挂载前缀：`/api/v2/users`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/v2/users` | `get_users_v2` | 登录态；门禁 role 过滤需 admin，否则需 admin/department | query `page,size,query,role` | 列出用户 |
| `GET` | `/api/v2/users/{user_id}/activities` | `get_user_activities_v2` | 登录态 | query `page,perpage,search,sort,regex,asc` | 用户活动列表；调用 validate_read_user_permission("volunteer") |
| `GET` | `/api/v2/users/{user_id}/time` | `get_user_time_v2` | 登录态 | query `allow_cache` | 用户时长；调用 validate_read_user_permission("volunteer") |
| `GET` | `/api/v2/users/{user_id}/time_statistics` | `get_user_time_statistics_v2` | 登录态 | path `user_id` | 用户时长百分位统计；调用 validate_read_user_permission("volunteer") |
