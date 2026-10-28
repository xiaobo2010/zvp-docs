# 封禁（`/api/v3/ban`）

> 来源：`routers/ban_router.py`、`routers/ban_violations_router.py`　·　挂载前缀：`/api/v3/ban`

封禁规则决定「什么证据、达到什么阈值、按什么梯度处罚」；设备封禁是同一套判定层的另一个作用面。所有读端点要求 `admin` 或对应角色开关（`ban.read`），所有写端点要求 `admin` + **密码再认证**。

## 端点

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/v3/ban/rules` | `list_ban_rules` | `admin` / `ban.read` | 无 | 规则清单（含内置标记与当前梯度） |
| `GET` | `/api/v3/ban/triggers` | `list_triggers` | `admin` / `ban.read` | 无 | 平台已注册的检查项（新增规则的下拉来源） |
| `POST` | `/api/v3/ban/rules` | `create_ban_rule` | `admin`、记日志、**再认证** | body `PostRuleRequest{code,name,trigger,enabled?,config?,penalties,verification_token}` | 新增规则（201） |
| `PUT` | `/api/v3/ban/rules/{code}` | `update_ban_rule` | `admin`、记日志、**再认证** | body `PutRuleRequest{name?,trigger?,enabled?,config?,penalties?,verification_token}` | 修改阈值 / 梯度 / 开关 |
| `DELETE` | `/api/v3/ban/rules/{code}` | `delete_ban_rule` | `admin`、记日志、**再认证** | body `{verification_token}`（DELETE 也走 body） | 删除规则（204） |
| `GET` | `/api/v3/ban/users` | `list_banned_users` | `admin` / `ban.read` | query `page,perpage,query,include_expired` | 当前生效的账号封禁 |
| `GET` | `/api/v3/ban/records` | `list_ban_records` | `admin` / `ban.read` | query `page,perpage,query` | 封禁流水 |
| `GET` | `/api/v3/ban/devices` | `list_banned_devices` | `admin` / `ban.read` | query `page,perpage,query,include_expired` | 当前生效的设备封禁 |
| `POST` | `/api/v3/ban/devices` | `ban_device` | `admin`、记日志、**再认证** | body `{xuehai_id,reason?,until?,verification_token}` | 手动封设备 |
| `DELETE` | `/api/v3/ban/devices/{xuehai_id}` | `unban_device` | `admin`、记日志、**再认证** | body `{verification_token}` | 解封设备（204；未封过 404） |
| `POST` | `/api/v3/ban/devices/{xuehai_id}/reset-gradient` | `reset_device_gradient` | `admin`、记日志、**再认证** | body `{verification_token}` | 清零设备处罚梯度（幂等 204） |

`ban_violations_router` 另提供违规证据的列表（`GET /violations`）与客户端上报入口（`POST /violations`），用于「为什么封他」的追溯。

## 密码再认证

所有写端点必须带 `verification_token`：由 `POST /api/users/auth` 的 `purpose=verify` 用密码换出的 **5 分钟**短令牌（scope=`temporary_token`）。缺失、过期、伪造或非本人一律：

```json
{ "detail": "需要密码验证后才能继续" }
```

状态码 `403`。`DELETE` 与 `POST /reset-gradient` 也走请求体（与附件/用户端点一致），因为要把令牌放进 URL 等于把它写进访问日志。

## 规则请求体

```json
{
  "code": "agent_jailbreak",
  "name": "agent 越狱与提示注入",
  "trigger": "agent_jailbreak",
  "enabled": true,
  "config": { "threshold": 3, "window_hours": 24 },
  "penalties": [7, 30, 90, null],
  "verification_token": "<5 分钟短令牌>"
}
```

| 字段 | 约束 |
|---|---|
| `code` | `^[a-z][a-z0-9_]{2,63}$`；全局唯一，冲突 409 |
| `trigger` | 必须是 `GET /triggers` 里注册的检查项；一个检查项只能被一条规则占用（否则「按哪条梯度封」取决于查询顺序），冲突 409；**创建后不可改**（PUT 传不同值 422） |
| `config.threshold` | 2–200 |
| `config.window_hours` | 1–168 |
| `penalties` | 1–6 档；每档 1–3650 天；严格升序；`null` 只允许出现在最后一位，表示「此后永久」 |

规则视图：`{code,name,trigger,trigger_name,enabled,builtin,config,penalties,updated_at}`。

## 立即封号语义（2026-10-29）

ZVAgent 的内容安全命中（`trigger=agent_jailbreak`）不再「首次只警告」：

- 判定成立 → 记 `ban_violations` 证据 → 按该规则 `penalties` 的**第一档立即封号**。第二次违规自然进第二档。
- 对外只回 `AI_SAFETY_REJECTED` 与固定文案，不暴露档位、天数或规则名。
- 同一份证据 5 分钟内去重；已在封禁中只记新证据，不重复落封。
- `admin` 在审核豁免名单内（自动封管理员会把平台锁死）；`department` 不在。
- 封禁/解封、梯度调整都写 `ban_change`（`warning`）审计，且该 `ltype` 在保留期保护集内。

## 设备封禁

`POST /devices` 的 `until` 省略或 `null` 即永久；客户端送来的带时区时间会先折成本地 naive 再落库（列是 `TIMESTAMP`，直接绑定时驱动会保留 UTC 墙钟，东八区下「封 24 小时」会被读成只剩 16 小时）。手动封设备**不动**设备梯度计数器。

`include_expired=1` 可把已刑满的行一起列出（行不会自动删，判定层用 `ban_is_active` 排除）。因账号封禁而「实时推导为封禁」的设备不在 `device_bans` 行里，账号解封后自动恢复。
