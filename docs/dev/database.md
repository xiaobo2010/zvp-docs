# 数据库

ZVP 平台的数据访问层采用**仓储模式**，统一入口为 `db/backends.py`。

## 后端

**Oracle Autonomous Database (ADB)**，通过 **SQLAlchemy 2.0 async + python-oracledb（thin 模式）** 访问。

- ADB 是**唯一后端**，没有开关，也没有非 ADB 的降级路径。
- 历史实现基于 MongoDB（Motor）；平台正由 MongoDB 迁移至 ADB，迁移过程见迁移记录。

## 双实例

系统区分 **prod / test** 两个逻辑实例，由 `ORACLE_ACTIVE`（`"prod"` / `"test"`）选择：

| 实例 | DSN | wallet 目录 | wallet 密码 |
|---|---|---|---|
| prod | `ORACLE_DSN` | `ORACLE_WALLET_DIR` | `ORACLE_WALLET_PASSWORD` |
| test | `ORACLE_TEST_DSN` | `ORACLE_TEST_WALLET_DIR` | `ORACLE_TEST_WALLET_PASSWORD` |

用户 / 密码可共享，但 **DSN、wallet、wallet 密码按实例区分**。

## 连接（thin 模式）

- python-oracledb **thin 模式要求 `ewallet.pem`**，不支持 SSO 的 `cwallet.sso`（那是 thick 模式）。
- 下载的 wallet 中 `ewallet.pem` 是**加密**的，因此 **`wallet_password` 必填**——缺失时驱动会退回交互式询问 PEM 口令。
- thin 模式下**必须同时提供 `config_dir` 与 `wallet_location`** 才加载 mTLS 客户端证书，仅传 `config_dir` 会报 `ORA-12506`。

## 目录结构

```
db/
├── backends.py        # 仓储工厂
├── models.py          # 数据模型
├── ids.py             # OID 归一化 / 校验
├── oracle/
│   ├── engine.py      # 引擎 / 连接
│   ├── session.py     # 会话管理
│   ├── config.py      # 连接配置解析
│   ├── ddl/           # 迁移脚本（幂等）
│   └── README.md
└── repos/
    ├── protocols.py   # 仓储接口
    └── oracle/        # Oracle 实现（users / groups / activities / members / changelog …）
```

## 迁移脚本

- DDL 迁移位于 `db/oracle/ddl/`，按序执行且**幂等**：对已存在的对象吞掉相应错误码（如 `ORA-01451`、`ORA-40664`），保证在既有 schema 与新 schema 上均可重复执行。
- 迁移前可用 `db/ddl_check.py` 做一致性检查。

!!! note
    数据库结构详情（表、字段、约束）以数据模型与 DDL 脚本为准；本页仅描述访问架构与连接约定。
