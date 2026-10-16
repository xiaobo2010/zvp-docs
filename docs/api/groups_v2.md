# 分组 v2（`/api/v2/groups`）

> 来源：`routers/groups_v2_router.py`　·　挂载前缀：`/api/v2/groups`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/v2/groups` | `get_groups_v2` | 登录态；门禁 admin/department | query `page,size,query` | 列出班级+权限组 |
| `GET` | `/api/v2/groups/{group_id}` | `get_group_v2` | 登录态；门禁 admin/department 或本人所在班级 | path `group_id` | 读取单个班级 |
| `GET` | `/api/v2/groups/{group_id}/activities` | `get_group_activities_v2` | 登录态 | query `page,perpage,search,sort,regex,asc` | 班级活动；调用 validate_read_group_permission("volunteer") |
| `GET` | `/api/v2/groups/{group_id}/time` | `read_users` | 登录态 | query `query,page,perpage,allow_cache,sort,asc` | 班级用户时长；调用 validate_read_group_permission("volunteer") |
| `GET` | `/api/v2/groups/{group_id}/users` | `get_group_users_v2` | 登录态 | query `page,perpage,search,sort,regex,asc,exceeding,shortage,pwdm,start,end` | 班级用户列表；调用 validate_read_group_permission("volunteer") |
| `GET` | `/api/v2/groups/{group_id}/statistics/compliance` | `stat_group_compliance` | 登录态 | path `group_id` | 班级合规统计；调用 validate_read_group_permission("volunteer") |
