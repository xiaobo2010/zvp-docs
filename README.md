# ZVP 平台开发文档

镇海中学义工管理平台（ZVMS / ZVP）的 API 与开发文档站点。

**在线访问：<https://xiaobo2010.github.io/zvp-docs/>**

## 内容

- 快速开始：本地拉起前后端
- 架构总览：分层、目录结构、路由版本策略
- API 参考：101 个端点的路径 / 方法 / 鉴权 / 参数 / 用途
- 开发指南：后端 · 前端 · 数据库

## 本地预览

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/mkdocs serve
```

## 发布

推送到 `main` 后由 GitHub Actions 自动构建并发布到 GitHub Pages（见 `.github/workflows/deploy.yml`）。

## 说明

本仓库仅存放文档站点，不含平台源码。
