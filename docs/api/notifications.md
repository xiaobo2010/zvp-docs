# 站内通知（`/api/v3/notifications`）

> 来源：`routers/notifications_router.py`　·　挂载前缀：`/api/v3/notifications`

通知有**全局版本**（客户端本地「已读 / 不再显示」标记按它推进）和**单条版本**（每次内容变化 +1）。管理员写操作全部要求 `admin`（或 `notify.publish` 角色开关）+ **密码再认证**。

## 端点

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/v3/notifications` | `read_notifications` | 登录态 | 无 | 当前有效通知 + 全局版本 |
| `GET` | `/api/v3/notifications/admin` | `read_notifications_admin` | `admin` / `notify.publish` | 无 | 管理列表（含停用） |
| `POST` | `/api/v3/notifications` | `create_notification` | `admin` / `notify.publish`、记日志、**再认证** | body `{title,content,verification_token}` | 创建即发送（201） |
| `PUT` | `/api/v3/notifications/{oid}` | `update_notification` | 同上、**再认证** | body `{title?,content?,active?,verification_token}` | 编辑 / 停用 / 恢复 |
| `DELETE` | `/api/v3/notifications/{oid}` | `delete_notification` | 同上、**再认证** | body `{verification_token}` | 硬删除（204） |
| `GET` | `/api/v3/notifications/{oid}/versions` | `read_notification_versions` | `admin` | 无 | 完整修订历史（含已删除） |
| `POST` | `/api/v3/notifications/{oid}/rollback` | `rollback_notification` | `admin` / `notify.publish`、**再认证** | body `{version,verification_token}` | 回滚到指定版本 |

## 密码再认证

创建、编辑、删除、回滚都要求请求体带 `verification_token`：由 `POST /api/users/auth` 的 `purpose=verify` 用密码换出的 **5 分钟**短令牌。缺失、过期、伪造或非本人：

```json
{ "detail": "需要密码验证后才能继续" }
```

状态码 `403`。`DELETE` 也走请求体，避免令牌进 URL / 访问日志。

## 请求与响应

创建：

```json
{
  "title": "标题（1–200 字符）",
  "content": "正文（1–4000 字符，UTF-8 ≤4000 字节）",
  "verification_token": "<5 分钟短令牌>"
}
```

成功 `201`：`{"status":"ok","code":201,"data":{"notification":{...},"version":43}}`。
`author_id` / `author_name` 由服务端从当前登录用户解析，不接受客户端传入。

编辑：`title` / `content` / `active` 至少给一个；未给的字段不变。单条版本与全局版本各 +1；`active=false` 是停用，用户端不再返回，管理端仍可见，可再用 `true` 恢复。

删除：硬删除，全局版本 +1；删除前先落一条 `op=delete` 快照，因此仍可回滚恢复（沿用原 oid）。通知不存在时 `404`，版本不变。

回滚：`{"version": 2, "verification_token": "..."}`。通知仍在则与编辑同路径（行锁 + 单条版本 +1）；已被删除则用原 oid 重新插入（版本 = 历史最大版本 + 1）。回滚只还原内容，发布人与 `created_at` 仍是不可变事实。

错误：`400`（`version` 非法 / oid 形状不对）、`403`（无权限 / 再认证失败）、`404`（通知或版本不存在）。

## 审计

创建 / 编辑 / 删除 / 回滚各写一条 `APP_LOGS`（`notification_create` / `notification_update` / `notification_delete` / `notification_rollback`）。通知版本快照保留 90 天（`notification_versions`），比通知行本身短——它只承担「回滚与追溯最近改动」，不是永久档案。
