# 后端

后端基于 **FastAPI**，数据访问用 **SQLAlchemy 2.0 async + python-oracledb**（thin 模式），运行在 Oracle Autonomous Database 上。

## 目录结构

```
zvms4-backend-python/
├── main.py            # 应用入口：中间件、异常处理、路由挂载、定时任务
├── config.py          # 业务常量（时长基准、折算比例、查询上限等）
├── settings.py        # 环境相关配置（不入库；由 settings.example.py 派生）
├── routers/           # HTTP 路由（按业务域 + 版本分层）
├── db/
│   ├── backends.py    # 仓储工厂（统一数据访问入口）
│   ├── models.py      # 数据模型
│   ├── ids.py         # OID 归一化与校验
│   ├── oracle/        # Oracle 连接与 DDL（engine / session / config / ddl）
│   └── repos/         # 仓储接口（protocols.py）与 Oracle 实现（oracle/*）
├── util/              # 鉴权、权限、证书、对外能力等
│   ├── object_id.py   # 令牌解析与当前用户依赖
│   └── permission/    # 权限判定
├── tasks/             # APScheduler 定时任务（统计重建、报表生成）
├── typings/           # 类型定义（日志、用户等）
└── tests/             # 测试
```

## 应用入口

`main.py` 负责：

- 创建 `FastAPI` 实例，统一响应类为 `JSONResponse`，内建 `docs_url=None`（不暴露默认文档页）
- 注册 CORS、请求加固等中间件
- 注册全局异常处理（校验错误等）
- 挂载各业务路由（见 [API 参考](../api/index.md#路由挂载)）
- 启动 / 关停钩子（数据库 ping、socket.io、APScheduler 调度器）

## 数据访问

- 一切数据访问经 `db/backends.py` 的仓储工厂获取实现，业务代码不直接持有连接。
- 仓储接口定义在 `db/repos/protocols.py`，Oracle 实现位于 `db/repos/oracle/`。
- 连接由 `db/oracle/engine.py` 与 `db/oracle/session.py` 管理；ADB 为**唯一后端**，无 fallback。
- 迁移脚本位于 `db/oracle/ddl/`（幂等，容忍既有对象已存在）。

## 鉴权依赖

| 依赖 | 语义 |
|---|---|
| `get_current_user` | 必须已认证；无效令牌抛 401 |
| `optional_current_user` | 匿名可访；未登录返回 `None` |
| `compulsory_temporary_token` | 要求短期令牌的敏感写操作 |

令牌为 JWT（`HS256`），签名密钥从 `aes_key.txt` 读取，解析逻辑见 `util/cert.py`。

## 测试

```bash
pytest
```

测试位于 `tests/`；离线用例隔离了外部依赖，可在无数据库连接的环境运行。
