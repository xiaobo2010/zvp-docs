# 活动 v2（`/api/v2/activities`）

> 来源：`routers/activities_v2_router.py`　·　挂载前缀：`/api/v2/activities`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `POST` | `/api/v2/activities` | `create_activity_v2` | 登录态、记日志 | body `Activity` | 创建活动；调用 volunteer.validate_create_permission (admin/department) |
| `GET` | `/api/v2/activities` | `get_activities_v2` | 登录态 | query `page,perpage,search,sort,asc,regex,activity_type` | 活动列表（分页/过滤） |
| `GET` | `/api/v2/activities/{activity_id}` | `get_activity_v2` | 登录态 | path `activity_id` | 获取单个活动 + 成员数 |
| `DELETE` | `/api/v2/activities/{activity_id}` | `delete_activity_v2` | 登录态、记日志 | path `activity_id` | 删除活动；调用 volunteer.validate_update_permission (admin/department) |
| `GET` | `/api/v2/activities/{activity_id}/members` | `get_activity_members_v2` | 登录态 | query `page,perpage,search,sort,asc,regex` | 获取活动成员列表 |
| `GET` | `/api/v2/activities/{activity_id}/members/{session_id}` | `get_activity_member_v2` | 登录态 | path `session_id` | 获取单个活动成员记录 |
| `POST` | `/api/v2/activities/{activity_id}/members` | `add_activity_member_v2` | 登录态、记日志 | body `ActivityMember` | 添加活动成员；调用 volunteer_member.validate_create_permission (admin/department) |
| `DELETE` | `/api/v2/activities/{activity_id}/members/{session_id}` | `delete_activity_member_v2` | 登录态、记日志 | path `session_id` | 删除活动成员记录；调用 volunteer_member.validate_update_permission (admin/department) |
| `PUT` | `/api/v2/activities/{activity_id}/status` | `modify_activity_status_v2` | 登录态、记日志 | body `PutActivityStatus{status}` | 审核/修改活动状态；调用 volunteer.validate_check_permission + 手动 `user.get("perm")` 检查（注: 疑似应写 `per`） |
| `POST` | `/api/v2/activities/amalgamation` | `amalgamate_activities_v2` | 短期令牌、记日志 | body `AmalgamationForm` | **（已移除）** 返回 410 |
| `PUT` | `/api/v2/activities/{activity_id}/members/{document_id}/record` | `update_user_duration_v2` | 登录态、记日志 | body `UpdateUserRecord{duration,mode}` | 更新成员时长/模式；调用 volunteer_member.validate_update_permission |
| `PUT` | `/api/v2/activities/{activity_id}/info` | `update_activity_info` | 登录态、记日志 | body `UpdateActivityInfo{name,description}` | 修改活动名称与描述；调用 volunteer.validate_update_permission |

!!! warning "待确认"
    清单标注：`routers/activities_v2_router.py` 的活动状态门禁写作 `user.get("perm")`（而其它端点用 `"per"`），疑为拼写不一致——现状照录，未修改源码，需人工确认。
