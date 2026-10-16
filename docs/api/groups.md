# 分组（`/api/groups`）

> 来源：`routers/groups_router.py`　·　挂载前缀：`/api/groups`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `POST` | `/api/groups` | `create_group` | 登录态、记日志；门禁 admin | body `Group` | 创建用户组 |
| `GET` | `/api/groups` | `get_groups` | 登录态；门禁 `type=permission` 需 admin/department，否则 403；`type=all` 对非特权静默降级为仅 class | query `page,perpage,search,type` | 获取全部组 |
| `GET` | `/api/groups/{group_id}` | `get_group` | 登录态 | path `group_id` | 获取单个组；调用 validate_read_group_permission("volunteer", allow_class=True) |
| `PUT` | `/api/groups/{group_id}/name` | `update_group_name` | 登录态、记日志；门禁 admin | body `PutGroupName{name}` | 修改组名 |
| `GET` | `/api/groups/{group_id}/activities` | `get_class_activities` | 登录态；门禁 admin/auditor/department/secretary同班 | query `page,perpage,query` | 获取组相关活动 |
| `GET` | `/api/groups/{group_id}/user` | `get_users_in_class` | 登录态；门禁 admin/auditor/department/secretary同班 | query `page,perpage,search,pwdm` | 获取班级用户 |
| `GET` | `/api/groups/{group_id}/time` | `get_user_times_in_class` | 登录态；门禁 admin/auditor/department/secretary同班 | query `page,perpage,exceeding,shortage,start,end,search,allow_cache` | 获取班级用户时长 |
| `PUT` | `/api/groups/{group_id}/description` | `update_group_description` | 登录态、记日志；门禁 admin | body `PutGroupDescription{description}` | 修改组描述 |
| `DELETE` | `/api/groups/{group_id}` | `delete_group` | 短期令牌、记日志；门禁 admin | path `group_id` | 删除用户组 |
| `GET` | `/api/groups/{group_id}/template` | `get_group_template` | 登录态；门禁 admin/auditor/department/secretary同班 | query `export_format` (`ExportFormat`) | 导出班级模板 |
