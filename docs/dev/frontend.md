# 前端

前端为 **Vue 3 + Vite + TypeScript** 单页应用，UI 基于 Element Plus，状态管理用 Pinia。

## 目录

```
zvms4-frontend/
├── src/
│   ├── api/          # 后端接口封装（按业务域分目录）
│   ├── components/   # 通用组件
│   ├── views/        # 页面
│   ├── router/       # Vue Router 路由与守卫
│   ├── stores/       # Pinia 状态
│   └── ...
├── public/
└── package.json
```

## 本地开发

```bash
npm install
npm run dev          # vite --port 5173
```

开发服务器默认监听 `5173`，需将 `/api/` 代理到后端（见 Vite 配置）。

## 常用脚本

| 命令 | 作用 |
|---|---|
| `npm run dev` | 启动开发服务器（5173） |
| `npm run build` | 类型检查 + 生产构建 |
| `npm run build-only` | 仅构建（跳过类型检查） |
| `npm run preview` | 预览生产构建（5173） |
| `npm run type-check` | `vue-tsc` 类型检查 |
| `npm run lint` | ESLint 自动修复 |
| `npm run format` | Prettier 格式化 |
| `npm run test:unit` | Vitest 单元测试 |
| `npm run test:e2e` | Cypress 端到端测试 |

## 约定

- 接口调用统一收敛在 `src/api/`，按业务域拆分（如 `activity/`、`groups/`）。
- 路由守卫负责登录态校验收敛；注意避免全局在飞请求的过度取消（历史上曾导致列表切换后空白）。
- PWA：站点带 Service Worker，改动后需注意旧客户端的即时接管。
