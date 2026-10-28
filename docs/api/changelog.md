# 更新日志（`/api/changelog`）

> 来源：`routers/changelog_router.py`　·　挂载前缀：`/api/changelog`

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/changelog` | `read_changelog` | 公开 | 无 | 读取版本记录列表 |
| `POST` | `/api/changelog` | `create_changelog` | 登录态、记日志；门禁 `_require_admin`、**再认证** | body `ChangelogEntryRequest{version,date,items,verification_token}` | 新增版本记录 |
| `PUT` | `/api/changelog/{oid}` | `update_changelog` | 同上、**再认证** | body `ChangelogEntryRequest` | 更新版本记录 |
| `DELETE` | `/api/changelog/{oid}` | `delete_changelog` | 同上、**再认证** | body `{verification_token}` | 删除版本记录 |

## 密码再认证

新增、修改、删除都要求请求体带 `verification_token`：由 `POST /api/users/auth` 的 `purpose=verify` 用密码换出的 **5 分钟**短令牌（scope=`temporary_token`）。缺失、过期、伪造或非本人：

```json
{ "detail": "需要密码验证后才能继续" }
```

状态码 `403`。`DELETE` 也走请求体，避免令牌进 URL / 访问日志。

## 说明

- `version` / `date` / `items` 是自由文本与字符串数组，服务端不做语义校验（版本号格式由管理员约定）。
- 更新日志已并入通知体系：管理员在「通知」页的「版本更新」卡片新增/修改条目后，后端自动以 SYSTEM 名义向用户下发版本公告。原独立 `/changelog` 页面与入口已移除。
- 写操作各记一条 `APP_LOGS`（`changelog_create` / `changelog_update` / `changelog_delete`）。
