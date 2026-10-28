# 架构总览

## 分层

```
┌─────────────────────────────────────────────┐
│  前端  Vue 3 + Vite + TS + Element Plus      │
│         · Pinia 状态 · Vue Router            │
└───────────────┬─────────────────────────────┘
                │  /api/  (JSON)  /socket.io/
┌───────────────▼─────────────────────────────┐
│  后端  FastAPI                               │
│   ┌────────────┐  ┌──────────┐  ┌─────────┐ │
│   │  routers/  │→ │  util/   │→ │   db/   │ │
│   │  路由/鉴权 │  │ 业务逻辑 │  │ 数据访问 │ │
│   └────────────┘  └──────────┘  └────┬────┘ │
└───────────────────────────────────────┼─────┘
                                        │
                            ┌───────────▼───────────────────┐
                            │  Oracle Autonomous Database    │
                            │  （python-oracledb thin 模式）  │
                            └───────────────────────────────┘
```

- **routers/**：HTTP 路由与请求/响应模型。按业务域拆分（users / groups / activities / logs / …），并按版本分层（见下）。
- **services/**：业务服务层。AI 网关与 **ZVAgent**（`services/ai/*`、`services/agent/*`）、文件系统原语、封禁与风控、通知与保留期任务等。
- **db/**：数据访问层。仓储（repository）模式，`db/backends.py` 为统一入口，当前实现为 Oracle ADB。
- **util/**：鉴权、权限判定、OID 校验、登录/读查询频控等横切能力。
- **tasks/**：APScheduler 定时任务，如时长统计重建、报表生成、保留期清理。
- **typings/**：日志与领域类型定义。

## ZVAgent 与 AI 网关

AI 能力统一以 **ZVAgent** 对外（「AI 网关」是 `/api/ai/*` 模块名）：

- **模型访问**：`services/ai/` 管理凭据、路由、额度、频控、熔断与计费；对外提供 `/api/ai/v1/*` OpenAI 兼容面。
- **会话与执行**：`ai_conversations` / `ai_messages` / `ai_runs` 持久化会话；`services/agent/run_queue.py` 单 worker 领取后台 run，逐轮落消息与计费。
- **工具**：封闭注册表（活动、文件、数据库、SSH、平台治理 / 运维），读工具直接执行，写工具生成提案并由用户确认；运维写工具另需另一位管理员的 6 位动态码。
- **只读场景**：终端 / 文件编辑的「问一句」走 `POST /api/ai/scenes/{scene}/runs`，工具裁到只读集、参考内容脱敏且不落消息表。
- **内容安全**：提交瞬间做规则 + LLM 灰区判定，**fail-closed**；判定成立记违规并按规则首档立即封号，admin 天然旁路。

详见 [ZVAgent / AI 网关](api/ai.md)。

## 安全基线

| 面 | 机制 |
|---|---|
| 登录 | 账号 5 次/分钟 + IP 60 次/分钟（`AUTH_RATE_LIMITED`）；登出轮换 `session_id` 让旧 token 立即失效 |
| 管理写操作 | 请求体 `verification_token`（5 分钟密码再认证短令牌）；缺失/无效 403 |
| 重查询 | 每用户 60 次/分钟读预算（`READ_RATE_LIMITED`），管理员同一档位 |
| 文件 | 系统文件与 agent `fs_*` 只允许 `/www/zvms4` 与 `/home`（越界 403 `FILES_ROOT_FORBIDDEN`），`/proc`/`/sys` 与凭据文件另拒 |
| 传输与浏览器 | 统一安全响应头 + CSP（无 `unsafe-eval`）、`X-Frame-Options: DENY`、HSTS |
| 请求门禁 | shield 中间件 418 拦截并采集命中原因 / referer / origin / XFF / UA；socket.io 只认 `Authorization` 头与 auth 载荷 |
| 审计 | `APP_LOGS` 记录 IP/UA；AI 调用流水零正文，`AI_PROMPT_AUDIT` 为受控正文例外（脱敏、限期、读取留痕） |

## 路由版本策略

后端按 URL 前缀分层提供 API：

| 前缀 | 说明 |
|---|---|
| `/api/…` | 基础层（v1）。用户、群组、日志、导入导出、附件、更新日志、系统 |
| `/api/ai/…` | AI 网关与 ZVAgent。OCR、OpenAI 兼容面、会话 / run、只读场景、管理面 |
| `/api/v2/…` | 第二层。活动、用户、群组、时长、MCP；统计接口亦挂载于此 |
| `/api/v3/…` | 第三层。用户 / 群组 / 活动的高阶写操作（建改角色、建班、成员进出）、封禁、通知 |

> 具体挂载前缀以 `main.py` 中 `include_router(..., prefix=...)` 为准，见 [API 参考](api/index.md)。

## 鉴权与权限模型

- **认证**：OAuth2 Bearer Token（`oauth2_scheme`）。依赖项 `get_current_user` 解析出当前用户；`optional_current_user` 用于匿名可访的端点；`compulsory_temporary_token` 用于需要临时令牌的写操作。
- **授权**：基于**权限组**（团队）的读写权限判定，辅以角色区分（学生 / 干事 / 管理员等）。部分列表端点还会做**降级处理**（非特权用户看到裁剪后的结果），而非直接 403。
- 权限判定的公共逻辑集中在 `util/permission/`。

## 关键设计约定

- **数据层迁移**：平台正由 MongoDB（历史实现）迁移至 Oracle ADB；ADB 为唯一后端，无降级路径。
- **日志 GET-only**：日志端点只读，杜绝篡改。
- **统计可重建**：`tasks/compute_time` 提供幂等的统计重建能力。
