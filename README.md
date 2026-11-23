# versionall · 健康训练营 H5（源码交付包）

> 仓库：BAOWEIDONG/versionall · 分支 `main` · 提交 `dd0167e`（2026-11-08）
> 交付内容：完整前端源码（不含 `node_modules`、`dist` 等依赖与构建产物）

## 1. 项目说明

健康训练营 H5，单页应用，覆盖学员端、教练端、营养师（运营）端三端，包含打卡、健康档案、积分排名、奖励兑换、营期管理、批注、结营报告、徽章、活动等模块。

技术栈：

- Vue 3 + TypeScript
- Vite 6
- Tailwind CSS 4（`@tailwindcss/vite`）
- Vant 4（移动端组件）
- Pinia（状态管理）
- vite-plugin-pwa（PWA / 离线缓存）

默认使用本地 mock 数据，**不依赖后端即可独立运行**；也预留了真实后端接口层（见第 6 节）。

## 2. 环境要求

- Node.js 20 及以上（本包已在 Node 22.23.4 验证）
- npm 10 及以上

## 3. 本地运行

```bash
npm install
cp .env.example .env
npm run dev
```

启动后访问终端提示的地址。默认部署基础路径是 `/versionall/`，因此本地地址为：

```
http://localhost:3000/versionall/
```

如果希望本地以根路径访问（`http://localhost:3000/`），把 `.env` 中的 `VITE_BASE` 改为 `/` 或留空后重新启动：

```bash
VITE_BASE=/
```

> `VITE_BASE` 是构建期变量，修改后需要重新执行 `npm run dev` / `npm run build`。

生产构建与本地预览：

```bash
npm run build
npm run preview
```

类型检查：

```bash
npm run lint
```

## 4. 测试账号

登录页选择对应角色，输入手机号，验证码填**任意 6 位数字**，勾选协议即可登录（mock 模式不校验验证码内容）。

| 角色 | 手机号 | 姓名 / 说明 |
|---|---|---|
| 学员 | `13800000001` | 李明 |
| 学员 | `13800000002` | 王丽 |
| 教练 | `13700000001` | 李教练 |
| 营养师 | `18888888888` | 管理员（登录页也有提示） |
| 营养师/督导 | `13900000009` | 运营督导（同时是体验教练/体验学员账号） |

更多账号见 `src/mock/data.ts` 中的 `MOCK_ACCOUNTS`。手机号必须是 `MOCK_ACCOUNTS` 中已配置且启用的账户，否则会提示“未在系统中配置”。

## 5. 构建与部署

`npm run build` 产物在 `dist/`，为纯静态文件，可直接放到任意静态服务器 / CDN。

部署路径由 `.env` 的 `VITE_BASE` 控制：

- GitHub Pages 子路径：`/versionall/`（默认）
- 域名根目录：`/` 或留空
- Nginx 子目录：如 `/sub/`

仓库自带 GitHub Actions 工作流 `.github/workflows/deploy.yml`：push 到 `main` 自动构建并发布 GitHub Pages。使用前需在仓库 Settings → Pages 中将 Source 设为 **GitHub Actions**。

## 6. 后端联调

切换 `.env`：

```bash
VITE_USE_MOCK=false
VITE_API_BASE=/api
```

- `VITE_USE_MOCK=true`：本地 mock 数据，独立演示（默认）。
- `VITE_USE_MOCK=false`：请求 `VITE_API_BASE` 指向的真实服务。

接口契约、请求封装和各方法注释集中在 `src/lib/api.ts`。当前约定的资源与端点包括：

- 登录：`POST /auth/login`
- 学员：`GET /students`
- 饮食记录：`GET/POST /diet-records`、`PATCH /diet-records/:id`
- 运动记录：`GET/POST /exercise-records`、`PATCH /exercise-records/:id`
- 体重记录：`GET/POST /weight-records`、`PATCH /weight-records/:id`
- 教练活动：`GET/POST /coach-activities`、`PATCH/DELETE /coach-activities/:id`
- 奖励档位：`GET/POST /reward-tiers`、`PATCH/DELETE /reward-tiers/:id`
- 奖励领取：`GET/POST /reward-claims`、`PATCH /reward-claims/:id`
- 指标配置：`GET/POST /metric-configs`、`PATCH/DELETE /metric-configs/:id`
- 营期：`GET/POST /camps`、`PATCH/DELETE /camps/:id`
- 账户：`GET/POST /accounts`、`PATCH/DELETE /accounts/:id`
- 积分商品：`GET/POST /point-products`、`PATCH/DELETE /point-products/:id`
- 积分兑换：`GET/POST /point-exchanges`、`PATCH /point-exchanges/:id`
- 人工加分：`POST /manual-score-records`
- 文件上传：`POST /upload`

约定日期格式为 `yyyy-MM-dd` 或 `yyyy-MM-dd HH:mm:ss`，接口返回 JSON，错误使用非 2xx 状态码。业务口径索引（连续打卡、积分、排名等）写在 `src/lib/api.ts` 文件头。

## 7. 本地数据与存储

纯前端 demo 的业务数据保存在浏览器 `localStorage`，刷新后仍在；清除浏览器站点数据即可恢复初始 mock 数据。

> 重要：`localStorage` 按域名共享、不按路径隔离。versionone / versiontwo / versionall 部署在同一域名下时会互相污染，因此本项目所有 key 均带 `versionall_` 前缀，统一由 `src/lib/storage.ts` 的 `nsKey()` 生成。新增 key 请继续使用 `nsKey()`，不要直接写字符串。

## 8. 目录结构

```
src/
├─ components/          页面与业务组件（学员/教练/营养师三端）
│  └─ ui/               通用 UI（按钮、卡片、导航、图表等）
├─ composables/         通用组合式函数
├─ lib/                 业务逻辑与基础设施
│  ├─ api.ts            前后端接口契约与请求封装
│  ├─ storage.ts        localStorage 命名空间
│  ├─ scoring.ts        积分规则
│  ├─ streak.ts         连续打卡与奖励解锁
│  └─ ...               报告、徽章、导出、图片压缩等
├─ mock/data.ts         mock 数据（账号、学员、营期、记录等）
├─ store/app.ts         Pinia 全局状态与视图路由
├─ types.ts             类型定义
├─ App.vue              视图分发
└─ main.ts              应用入口
public/                 PWA 图标
docs/                   产品方案文档
```

项目未使用 vue-router，页面切换由 `src/store/app.ts` 的 `view` 状态驱动，`src/App.vue` 负责按 `view` 渲染对应页面。

## 9. 交付前验证记录

在干净环境（Node 22.23.4 / npm 10.9.9）执行：

- `npm install`：通过
- `npm run build`：通过，生成 `dist/`（含 PWA `sw.js`，约 9 秒）
- `npm run lint`（`vue-tsc --noEmit`）：通过，无类型错误
- `npm run dev`：通过，`http://localhost:3000/versionall/` 返回 200，入口模块正常加载

## 10. 注意事项

- `index.html` 中引入了外部脚本 `https://baoweidong.github.io/pku-console/assets/pku-bridge.js`，用于嵌入 pku-console 时与父页面通信。独立运行不依赖它；如部署环境无法访问外网，可移除该行，不影响主流程。
- 项目开启了 PWA 缓存。调试时如果页面没更新，请硬刷新或在浏览器中注销 Service Worker。
- 图片 / 视频打卡数据存放在 `localStorage`，大文件可能触发浏览器配额限制，正式接入后端时应改为上传接口。
- 交付包不包含 `.env`、`node_modules`、`dist`。首次运行请按第 3 节复制 `.env.example`。
