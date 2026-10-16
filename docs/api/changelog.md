# 更新日志（`/api/changelog`）

> 来源：`routers/changelog_router.py`　·　挂载前缀：`/api/changelog`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/changelog` | `read_changelog` | 公开 | 无 | 读取版本记录列表 |
| `POST` | `/api/changelog` | `create_changelog` | 登录态、记日志；门禁 _require_admin | body `ChangelogEntryRequest{version,date,items}` | 新增版本记录 |
| `PUT` | `/api/changelog/{oid}` | `update_changelog` | 登录态、记日志；门禁 _require_admin | body `ChangelogEntryRequest` | 更新版本记录 |
| `DELETE` | `/api/changelog/{oid}` | `delete_changelog` | 登录态、记日志；门禁 _require_admin | path `oid` | 删除版本记录 |
