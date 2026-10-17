# 系统（`/api/system/maintenance`）

> 来源：`routers/system_router.py`　·　挂载前缀：`/api/system/maintenance`

维护模式（maintenance mode）的状态查询与开关。状态读写在本页路由；实际的拦截逻辑在应用中间件层（`main.py`），见下方「维护模式语义」。

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `GET` | `/api/system/maintenance` | `read_maintenance_state` | 公开 | 无 | 读取维护状态 |
| `POST` | `/api/system/maintenance/enable` | `enable_maintenance` | 登录态、记日志；门禁 `_require_system_admin` | body `EnableRequest{reason,announcement?}` | 开启维护模式 |
| `POST` | `/api/system/maintenance/disable` | `disable_maintenance` | 登录态、记日志；门禁 `_require_system_admin` | body `DisableRequest{reason?}`（请求体可省略） | 关闭维护模式 |

## 状态查询

公开端点，**无需鉴权**，维护模式开启期间仍可访问（供前端展示维护页）。

```json
{
  "status": "ok",
  "code": 200,
  "data": {
    "enabled": true,
    "since": 1729100000.0,
    "announcement": "系统升级维护，预计 22:00 恢复"
  }
}
```

- `enabled`：当前是否处于维护模式。
- `since`：本次开启的时刻（Unix 时间戳，秒，浮点）；未开启或已关闭时为 `null`。
- `announcement`：对外公告文案，可为 `null`。
- **不返回 `reason`**——内部原因刻意不外泄（见下）。

## 开启 / 关闭

开启请求体：

```json
{ "reason": "数据库升级", "announcement": "预计 22:00 恢复" }
```

| 字段 | 类型 | 约束 |
|---|---|---|
| `reason` | string | **必填**，1–200 字符 |
| `announcement` | string \| null | 可选，≤300 字符 |

关闭请求体可省略，或传可选 `reason`（≤200 字符）。

- **鉴权**：`_require_system_admin` —— 要求**严格 `admin`**（不含 `department`），并**即时查库复核**当前用户（不信 token 里的 `per` 快照）；用户不存在或已被封禁均 `403`。
- **副作用**：状态落库后，`enable` 会**立即断开所有已建立的 socket.io 连接**（前端不会自行掉线）。断开失败不影响 `enable` 返回。
- **响应**：与状态查询同结构，但在公开字段之外**额外返回 `reason`**（调用方能拿到自己刚写入的原因）。
- **幂等**：重复 `enable` 不会改写 `since`；重复 `disable` 仍会追写一条审计事件。

## 维护模式语义

开启后，**凡命中 `/api/**` 的业务请求一律 `503`**：

```json
{ "detail": "系统维护中，请稍后再试", "maintenance": true }
```

豁免（维护期间仍放行）：

| 放行项 | 说明 |
|---|---|
| `/api`、`/api/` | 根路径 |
| `/api/users/auth` | 登录端点 |
| `/api/cert` | 证书 |
| `/api/version` | 版本 |
| `/api/health` | 健康检查 |
| `/api/system*` | 整个前缀；管理员必须能在维护期间关闭维护模式 |
| 所有 `OPTIONS` 请求 | CORS 预检 |

**无角色豁免**：admin 对普通业务端点在维护期间同样 `503`——管理员只能走 `/api/system/*` 或上述豁免路径。

## 实时连接（socket.io）

- `/socket.io` 在维护期间**握手即被拒**：HTTP 握手返回 `503`，响应体带维护标记：

```json
{ "detail": "系统维护中，请稍后再试", "maintenance": true, "socket": true }
```

- websocket 升级请求在 `accept` 前被服务端关闭（表现为 HTTP `403`），不会建链。
- 开启维护模式的那一刻，`enable` 主动断开已建立的连接；关闭后新连接恢复。

## 状态存储

维护状态存在**本地 SQLite**（`db/system_state.py`，标准库 `sqlite3`），与业务库解耦——业务库不可用时仍能读写维护状态。路径由环境变量 `ZVMS_SYSTEM_DB` 覆盖，默认落在后端根目录的 `system_state.db`。**不在 ADB**。

读路径带约 **2 秒**进程内 TTL 缓存（`is_enabled()`）；socket.io 握手准入（`MaintenanceSocketGuard`）与 `connect` 兜底走**绕过缓存**的直读 `is_enabled_fresh()`，保证刚开启即刻生效。

## 运维注意（盾牌中间件）

`main.py` 的 `shield_middleware` 对 **`User-Agent` 不含 `Mozilla`** 的请求直接返回 `418`（`"请求已被拦截"`），且该中间件在最外层、优先于维护模式。用 `curl` / 脚本冒烟调试时**必须带 `-A "Mozilla/5.0"`**，否则会把 `418` 误判成业务失败：

```bash
curl -A "Mozilla/5.0" https://<host>/api/system/maintenance
```

## 演进预留

`_require_system_admin`（`routers/system_router.py`）是未来**双人授权**（在线批准）的唯一挂点。规划中的 `/api/system/otp` 与 maintenance 同形——每个子系统占一个资源段。
