# 快速开始

本文档帮助你在本地拉起 ZVP 平台的前后端开发环境。

## 前置条件

| 项 | 版本 |
|---|---|
| Python | 3.11+ |
| Node.js | 18+ |
| 数据库 | Oracle Autonomous Database（ADB）实例 + wallet |

## 后端

```bash
cd zvms4-backend-python

# 1. 安装依赖
pip install -r requirements.txt

# 2. 准备配置
cp settings.example.py settings.py
#    填写：SECRET_KEY、ORACLE_USER / ORACLE_PASSWORD、
#    ORACLE_DSN / ORACLE_WALLET_DIR / ORACLE_WALLET_PASSWORD
#    （test 实例对应 ORACLE_TEST_*，由 ORACLE_ACTIVE 选择）

# 3. 准备 Oracle wallet（下载的 wallet 含加密 ewallet.pem，thin 模式必需）
#    将 wallet 目录解压到 ORACLE_WALLET_DIR

# 4. 准备 JWT 签名密钥文件 aes_key.txt（util/object_id.py 从其读取 SECRET_KEY）

# 5. 启动
uvicorn main:app --reload
```

!!! warning "凭证不要入库"
    `settings.py`、`aes_key.txt`、wallet 文件、`*.pem` 均属敏感凭证，**不要提交到版本库**。

## 前端

```bash
cd zvms4-frontend
npm install
npm run dev          # http://localhost:5173
```

需将开发服务器上的 `/api/` 代理到后端地址（见 Vite 配置）。

## 下一步

- 了解分层与路由版本策略：[架构总览](architecture.md)
- 查接口：[API 参考](api/index.md)
- 后端 / 前端 / 数据库细节：[开发指南](dev/backend.md)
