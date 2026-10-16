# ZVP 平台开发文档

镇海中学义工（志愿服务）管理平台 —— **ZVMS / ZVP** 的 API 与开发文档。

## 这是什么

ZVP 面向中学的义工管理业务，覆盖：

- **义工活动**：活动创建、报名与审核、状态流转、时长核算
- **成员与权限**：用户、班级、权限组（团队）的建改查与成员进出
- **时长统计**：达标/差额核算、报表生成
- **附件与文件**：上传、下载、设备门禁
- **日志审计**：全站行为记录与检索
- **数据导入 / 导出**：批量数据迁移与报表导出

## 技术栈

| 层 | 技术 |
|---|---|
| 后端 | FastAPI + SQLAlchemy 2.0 async + Pydantic |
| 前端 | Vue 3 + Vite + TypeScript + Element Plus + Pinia |
| 数据库 | Oracle Autonomous Database（ADB，python-oracledb thin 模式）；历史实现为 MongoDB |
| 实时 | python-socketio（可选）+ APScheduler（定时统计任务） |

## 文档导航

<div class="grid cards" markdown>

- **[快速开始](getting-started.md)** —— 拉起本地前后端开发环境
- **[架构总览](architecture.md)** —— 分层、目录结构、路由版本策略
- **[API 参考](api/index.md)** —— 端点清单、鉴权、错误约定
- **[开发指南](dev/backend.md)** —— 后端 / 前端 / 数据库

</div>

!!! info "关于本仓库"
    本仓库只存放**文档站点**，不含平台源码。源码位于私有开发仓库，上游基准为开源项目 **zvms**（`zvms/zvms4-backend-python` + `zvms/zvms4-frontend`）。
