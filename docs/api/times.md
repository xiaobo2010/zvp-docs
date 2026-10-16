# 时长 v2（`/api/v2/times`）

> 来源：`routers/times_router.py`　·　挂载前缀：`/api/v2/times`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/v2/times` | `read_times` | 登录态 | query `query,page,perpage,sort,asc` | 按角色范围查询时长（admin/department 全部，secretary 本班，其他仅自己） |
