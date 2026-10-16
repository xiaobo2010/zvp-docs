# 导入导出（`/api/exports`、`/api/imports`）

> 来源：`routers/exports_router.py`、`routers/imports_router.py`　·　挂载前缀：`/api/exports`、`/api/imports`

## 导出 /api/exports

> 来源：`routers/exports_router.py`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `POST` | `/api/exports/users` | `export_users` | 登录态、记日志；门禁 admin | 无 (BackgroundTasks) | 发起用户列表导出任务 |
| `POST` | `/api/exports/time` | `export_time` | 登录态、记日志；门禁 admin | body `CreateExport{start,end,format,allow_cache,include_description}` | 发起时长报表导出任务 |
| `GET` | `/api/exports/{task_id}` | `get_export` | 登录态；门禁 admin | path `task_id` | 查询导出任务状态 |
| `POST` | `/api/exports/reports` | `request_download_reports` | 短期令牌、记日志；门禁 admin | 无 | 申请报告打包下载授权 |
| `GET` | `/api/exports/reports/download` | `download_reports` | token 校验（无登录态依赖）；手动校验: base64 token + 有效期 | query `token` | 下载报告压缩包 |
| `GET` | `/api/exports/{task_id}/file` | `get_export_file` | 登录态、记日志；门禁 admin | query `language` | 下载导出文件 |
| `GET` | `/api/exports` | `get_export_list` | 登录态；门禁 admin | 无 | 列出导出任务 |

## 导入 /api/imports

> 来源：`routers/imports_router.py`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `POST` | `/api/imports/activities` | `upload_activity_excel` | 登录态、记日志 | query `name,desc` + multipart `payload: UploadFile` | 上传活动 Excel 导入；**未见显式 admin 门禁**（仅登录态 + 名称/描述/格式校验） |

!!! warning "待确认"
    清单标注：`routers/imports_router.py` 的 `upload_activity_excel` 除登录态外**未见 per/admin 门禁**——如需文档标注权限，此为需人工确认点。
