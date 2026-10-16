# 统计 v2（`/api/v2/statistics/activities`）

> 来源：`routers/activity_statistics_router.py`　·　挂载前缀：`/api/v2/statistics/activities`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/v2/statistics/activities/{activity_id}/description` | `activity_issuance_description` | 登录态；门禁 _require_statistics_reader (admin/department/secretary) | path `activity_id` | 活动时长描述性统计 |
| `GET` | `/api/v2/statistics/activities/{activity_id}/layers` | `activity_issuance_layers` | 登录态；门禁 _require_statistics_reader (admin/department/secretary) | path `activity_id` | 活动时长分层 |
| `GET` | `/api/v2/statistics/activities/{activity_id}/analysis` | `activity_issuance_analysis` | 登录态；门禁 _require_statistics_reader (admin/department/secretary) | path `activity_id` | 活动直方图/分组/描述性分析 |
| `GET` | `/api/v2/statistics/activities/grades/{grade}/percentiles` | `get_time_percentiles` | 登录态；门禁 _require_statistics_reader (admin/department/secretary) | path `grade` | 年级时长百分位 |
| `GET` | `/api/v2/statistics/activities/grades/{grade}/indicators` | `get_time_indicators` | 登录态；门禁 _require_statistics_reader (admin/department/secretary) | path `grade` | 年级时长指标 |
