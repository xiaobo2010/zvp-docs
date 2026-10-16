# 活动 v3（`/api/v3/activities`）

> 来源：`routers/activities_v3_router.py`　·　挂载前缀：`/api/v3/activities`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/v3/activities` | `get_activities_v3` | 登录态；门禁 _is_volunteer_manager (admin/department) | query `page,size,query,type,status` | 列出活动 |
| `POST` | `/api/v3/activities` | `create_activity_v3` | 登录态、记日志；门禁 _is_volunteer_manager (admin/department) | body `CreateActivityRequest{name,description,type}` | 创建活动 |
| `GET` | `/api/v3/activities/{activity_id}/members` | `get_activity_members_v3` | 登录态 | query `page,size,query` | 活动成员列表 |
| `POST` | `/api/v3/activities/{activity_id}/members` | `add_activity_member_v3` | 登录态、记日志；门禁 _is_volunteer_manager (admin/department) | body `list[NewActivityMember]` | 批量添加/更新活动成员 |
