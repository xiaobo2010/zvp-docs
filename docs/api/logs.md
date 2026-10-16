# 日志（`/api/logs`）

> 来源：`routers/logs_router.py`　·　挂载前缀：`/api/logs`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/logs` | `read_logs` | 登录态；门禁 admin | query `performer,page,perpage,query,type,level,time_start,time_end,xuehai_id` | 读取系统日志 |
