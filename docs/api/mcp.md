# MCP v2（`/api/v2/mcp`）

> 来源：`routers/mcp_router.py`　·　挂载前缀：`/api/v2/mcp`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/v2/mcp/activities` | `mcp_get_activities` | 登录态；门禁 admin | query `page,perpage,search,activity_type,sort,asc` | MCP 只读：活动列表 |
| `GET` | `/api/v2/mcp/activities/{activity_id}` | `mcp_get_activity_details` | 登录态；门禁 admin | path `activity_id` | MCP 只读：活动详情 |
| `GET` | `/api/v2/mcp/activities/{activity_id}/statistics` | `mcp_get_activity_statistics` | 登录态；门禁 admin | path `activity_id` | MCP 只读：活动聚合统计 |
| `GET` | `/api/v2/mcp/activities/types/summary` | `mcp_get_activity_types_summary` | 登录态；门禁 admin | 无 | MCP 只读：按类型汇总 |
| `GET` | `/api/v2/mcp/activities/search/suggestions` | `mcp_get_activity_search_suggestions` | 登录态；门禁 admin | query `query,limit` | MCP 只读：活动名搜索建议 |
