# TypeScript 轨：层候选、硬约束、生态惯例

## 约束的两层分类

判定标准：**有没有官方文档能证实**。

- **技术硬约束**：官方文档明确说明不兼容、有版本耦合或平台限制。违反它做不出能跑的东西。
- **生态惯例**：某个框架、脚手架或社区的常见搭配，不是不可逾越的限制。跟着走省事，不走也能做。

> ⚠️ 参考 `vibestart` 之类的脚手架矩阵时尤其容易犯的错：它那份 19 条组合约束经逐条对官方文档审计后，**多数是脚手架的集成偏好，不是技术不兼容**。详见 `docs/adr/0002-constraint-layers.md`。

### 被官方文档推翻的常见约束

| 常被当成硬约束 | 实际 | 官方来源 |
|---|---|---|
| MongoDB 只能用 Mongoose | Prisma 官方有 MongoDB connector | https://docs.prisma.io/docs/orm/v7/core-concepts/supported-databases/mongodb |
| SQLite 只能用 Drizzle | Prisma 官方有 SQLite connector | https://www.prisma.io/docs/orm/core-concepts/supported-databases/sqlite |
| OpenAPI 风格必须配 Hono | oRPC 官方自带 OpenAPI Specification / Handler / Routing / Plugins 一整套 | https://orpc.unnoq.com/docs/openapi/openapi-specification |
| Bun 只能配 Hono | Bun 官方定位是通用运行时，能跑多种框架 | https://bun.com/docs |
| Electron 只能配纯前端 SPA | Electron 官方定位是通用桌面壳，加载何种渲染器由项目决定 | https://www.electronjs.org/docs |
| Workers 的后端只能是某几种 | Cloudflare Workers 官方支持多种框架与 handler | https://developers.cloudflare.com/workers/ |

### 查不到官方声明的（标「未证实」，不要当硬约束用）

- Effect RPC 是否必须配 Effect HTTP
- Effect HTTP 是否只能配 SPA
- TanStack Start 是否不支持 Electron
- oRPC、Workers 的穷举式后端限制

## 层候选

每个候选附官方定位原文。原文是给用户看的白话锚点。

| 层 | 候选与官方定位 |
|---|---|
| 前端框架 | **React** — "The library for web and native user interfaces"（https://react.dev/）<br>**Next.js** — "Next.js is a React framework for building full-stack web applications."（https://nextjs.org/docs）<br>**TanStack Start** — "A full-stack framework for React and Solid. Build with type-safe routing, server-side rendering, streaming, and server functions, powered by TanStack Router."（https://tanstack.com/start/latest） |
| 路由 | **TanStack Router** — "The route tree is the application contract. Define routes once, then let TypeScript carry paths, params, search state, loaders, links, and boundaries through every navigation."（https://tanstack.com/router/latest） |
| 后端 | **Hono** — "a small, simple, and ultrafast web framework built on Web Standards. It works on any JavaScript runtime: Cloudflare Workers, Fastly Compute, Deno, Bun, Vercel, Netlify, AWS Lambda, Lambda@Edge, and Node.js."（https://hono.dev/）<br>**框架自带 server** — Next.js / TanStack Start 自带，此时不再单选后端层 |
| API 风格 | **oRPC** — "Build end-to-end typesafe APIs and serve them over RPC, OpenAPI, and MCP"（https://orpc.dev/） |
| 数据层 · ORM | **Drizzle ORM** — "a lightweight and performant TypeScript ORM with developer experience in mind."（https://orm.drizzle.team/）<br>**Prisma ORM** — "Type-safe data layer — A declarative, type-safe schema rebuilt in native TypeScript"（https://www.prisma.io/） |
| 数据层 · 数据库引擎 | 见 `database-engines.md`（TS 轨与 Python 轨共用） |
| 认证 | **Better Auth** — "a framework-agnostic authentication and authorization framework for TypeScript"（https://www.better-auth.com/）<br>**Auth.js** — "a complete open source authentication solution for web applications"（https://authjs.dev/）<br>**Supabase Auth** — "makes it easy for end users to sign in to your application"（https://supabase.com/docs/guides/auth） |
| UI | **Tailwind CSS v4** — "A utility-first CSS framework packed with classes like flex, pt-4, text-center"（https://tailwindcss.com/）<br>**shadcn/ui** — "The Foundation for your Design System. Composable, accessible components with thoughtful defaults."（https://ui.shadcn.com/） |
| 构建与工具链 | **Vite** — "The Build Tool for the Web."（https://vite.dev/）<br>**Vite+** — "the unified toolchain and entry point for web development."；"neither a framework, nor a package manager and *definitely not a replacement for Vite*"（https://viteplus.dev/guide/）<br>**Turbopack**（Next.js 内置）— "an incremental bundler optimized for JavaScript and TypeScript, written in Rust, and built into Next.js"（https://nextjs.org/docs/app/api-reference/turbopack） |
| 测试 | **Vitest** — "A Vite-native testing framework."（https://vitest.dev/）<br>**Playwright** — "Fast and reliable end-to-end testing for modern web apps"（https://playwright.dev/） |
| 校验 | **Zod** — "TypeScript-first schema validation with static type inference"（https://zod.dev/）<br>**Valibot** — "The open source schema library for TypeScript with bundle size, type safety and developer experience in mind."（https://valibot.dev/）<br>**ArkType** — "TypeScript's 1:1 validator, optimized from editor to runtime"（https://arktype.io/） |
| 运行时 | **Node.js** — 官方要求生产用 LTS："Production applications should only use *Active LTS* or *Maintenance LTS* releases."（https://nodejs.org/en/about/previous-releases）<br>**Bun**、**Cloudflare Workers** — 见硬约束节的运行时耦合 |
| 部署 · 打包 | **Docker** — 官方 Node 镜像："The official Node.js docker image, made with love by the node community."，变体 `node:<version>` / `node:lts` / `node:alpine` / `node:slim`（https://github.com/nodejs/docker-node） |
| 部署 · 进程与反向代理 | **进程管理**（systemd 等）— Node 官方生产指南涉及用进程管理器处理崩溃重启（https://nodejs.org/en/docs/guides/）<br>**Nginx** — "a free, open-source, high-performance HTTP server and reverse proxy"（https://nginx.org/en/）<br>**Caddy** — "a powerful, enterprise-ready, open source web server with automatic HTTPS written in Go"（https://caddyserver.com/） |

## 技术硬约束

### 运行时版本下限

包清单里的 `engines` 与 `peerDependencies` 是硬约束。**具体数值会漂移，用前现查**（见 `version-verification.md`）。以下是 2026-11-04 实测的形态示例，用于说明查哪些字段：

| 包 | 字段与实测值 |
|---|---|
| vite | `engines.node: "^20.19.0 || >=22.12.0"` |
| next | `engines.node: ">=20.9.0"` |
| vitest | `engines.node: "^22.12.0 || ^24.0.0 || >=26.0.0"`，`peerDependencies.vite: "^6.4.0 || ^7.0.0 || ^8.0.0"` |
| @tanstack/react-router | `engines.node: ">=20.19"` |
| @tanstack/react-start | `engines.node: ">=22.12.0"`，`peerDependencies.vite: ">=8.0.0"` |
| @orpc/server | 官方 Runtime Requirements：Node.js 最低 18、推荐 22+（https://orpc.dev/docs/requirements） |

### TypeScript 版本

- npm 上 `typescript` 的 `latest` 已是 **TS 7**（Go 原生移植），官方说明："TypeScript 7.0 is made to be compatible with TypeScript 6.0's type-checking and command-line behavior."（https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/）
- **TS 7 不带 API**："it does not ship with an API. We expect TypeScript 7.1 to ship with a new (and different) API"。依赖 TS compiler API 的工具（如 typescript-eslint）需并排使用 `@typescript/typescript6`。
- **Zod 的 TS 下限**："Zod is tested against *TypeScript v5.5* and later. Older versions may work but are not officially supported."（https://zod.dev/）

### React 版本

- React 官方当前版本线为 19.x，18.x 归档在 18.react.dev（https://react.dev/versions）
- Next.js `peerDependencies.react`: `^18.2.0 || 19.0.0-rc-de68d2f4-20241204 || ^19.0.0`
- TanStack react-router / react-start `peerDependencies.react`: `>=18.0.0 || >=19.0.0`
- React Router (Remix) v8 `peerDependencies.react`: `>=19.2.7`
- Next.js App Router 使用内置 React canary："App Router: Uses React canary releases built-in… Pages Router: Uses the React version installed in your project's package.json."（https://nextjs.org/docs）

### Node 版本与 LTS

- 生产只用 Active LTS 或 Maintenance LTS（https://nodejs.org/en/about/previous-releases）
- 现行 LTS 线与 EOL 线逐季变动，**选型时现查**
- 官方发布周期正在变："Starting with Node.js 27, the release cycle will be annual and every major version will move to *LTS* status after its six-month *Current* phase"（同上）

### 生成代码先于类型检查

TanStack Router 的路由树由构建生成。官方原文："This file is automatically generated when you run TanStack Start (via `npm run dev` or `npm run start`)."（https://tanstack.com/start/latest/docs/framework/react/guide/routing）；路径字符串由 bundler 插件或 router CLI 自动管理。

**后果**：全新工程直接跑类型检查会报「找不到生成文件」。先构建再检查。

### 迁移顺序

**Drizzle**：`drizzle-kit generate --name=init` 生成 SQL，`drizzle-kit migrate` 读取迁移目录、比对已应用记录、执行并写入 `__drizzle_migrations` 表（https://orm.drizzle.team/docs/drizzle-kit-migrate）。迁移目录初始为空时，依赖迁移的集成测试会先失败——先 generate。

**Prisma**：**v7 起 `migrate dev` 不再自动触发 `prisma generate`**，需显式执行（https://www.prisma.io/docs/cli/v7/migrate/dev.md）。v6 则自动触发。跨大版本时这会静默打断流程。

### Next.js 上 Cloudflare Workers

官方路径存在，但不是零配置全特性：Cloudflare 官方推荐 **vinext** 而非 OpenNext 用于新项目，"OpenNext adapts the output of `next build` so it can run on different platforms, including Cloudflare Workers."（https://developers.cloudflare.com/workers/framework-guides/web-apps/opennext/）。Next.js 官方部署矩阵为 Node.js server / Docker container / Static export / Adapters（varies）。

**结论**：Workers 上跑 Next.js 属于「官方有路径但特性因适配器而异」，选型时要明确验证目标特性。

### 浏览器测试的下载成本

Playwright 官方："Each version of Playwright needs specific versions of browser binaries to operate. You will need to use the Playwright CLI to install these browsers."（https://playwright.dev/docs/browsers）。`npx playwright install` 会下载浏览器二进制（约百 MB 量级）。

若项目已有自带 Chromium 的浏览器工具（如 ego-browser），用它做人工与半自动验证可省掉这份下载；引入 Playwright 应是独立决策。

## 生态惯例（非强制）

- **TanStack Router + Vite/React 是 SPA 的常见底座**，TanStack Start 在需要 SSR/服务端函数时更合适。惯例，非限制。
- **shadcn/ui 现以 Base UI 为默认组件库**："Base UI is the default component library in shadcn/ui… Radix is still fully supported."（https://ui.shadcn.com/docs/changelog/2026-07-base-ui-default）。选 Base UI 还是 Radix 是惯例。
- **oRPC 的 adapter 覆盖面广**：官方列出 Fetch API、Node HTTP、Fastify、Elysia、Express、H3、Hono、Next.js、Nuxt、SvelteKit、TanStack Start、Cloudflare Workers、Electron、Expo 等（https://orpc.dev/docs/adapters）。后端层选谁，oRPC 都能接——这是 oRPC 的优势，不是对后端层的限制。
- **Better Auth 需要数据库存 users / sessions**："Better Auth connects to a database to store data."（https://www.better-auth.com/docs/concepts/database）。也支持无数据库的 stateless session。session 默认 7 天过期、1 天刷新（https://www.better-auth.com/docs/concepts/session-management）。**有数据库而无认证层时，生成的接口是公开的**——这是需要用户知晓的后果，不是框架限制。
- **Vite+ 通过别名接管 vite**：项目里 vite 指向 vite-plus 的核心包，且需与 vite-plus 版本一致，否则报版本解析失败。这是工具链的具体配置纪律，查 `viteplus.dev/guide`。
- **Vite+ 精确锁定 oxlint / oxfmt / vitest 版本**，升这些只能升 Vite+ 本身。这是工具链内部耦合，不是生态通例。

## 层之间怎么搭

给用户推荐组合时按这套骨架，缺层就明说「这层不需要」而不是留空：

```
前端框架 → 路由 → 后端 → API 风格 → 校验 → 数据库引擎 → ORM → 认证 → UI → 构建与工具链 → 测试 → 运行时 → 部署(打包 / 进程与反向代理)
```

两种常见形状：

**形状 A · SPA + 独立后端**
前端是纯浏览器应用，后端独立进程，之间走 HTTP。适合前后端分离部署、多个客户端共用一套 API。Hono + oRPC + Drizzle/Prisma 属于这一类。

**形状 B · 全栈框架自带 server**
Next.js 或 TanStack Start 内部就有服务端，不单选后端层，API 风格由框架约定。适合单体全栈应用，部署单元少一个。

两者的关键差异不是「谁更好」，而是**部署单元数量**与**后端逻辑的归属位置**。这一条要在决策表里写明。

### 选框架会连带锁住工具链

选了 Next.js，构建层就不是 Vite 而是内置的 Turbopack（官方：`--turbopack` 为默认，`--webpack` 可回退，https://nextjs.org/docs/app/api-reference/cli/next）。连带影响：

- **单元测试运行器**：Vitest 的 `peerDependencies.vite` 覆盖 `^6.4.0 || ^7.0.0 || ^8.0.0`，不与 Next.js 共存。Next.js 官方测试指南给出 Jest 与 Vitest 两套设置方式（https://nextjs.org/docs/app/guides/testing），E2E 推荐 Playwright。选 Next.js 时单测运行器要单独定。
- **类型检查**：Next.js 有独立的 `next typegen` 生成路由类型，官方说明可在构建前独立生成路由类型做类型检查（https://nextjs.org/docs/app/api-reference/cli/next）。
- **部署形态**：Next.js 官方列出 Vercel（推荐）、Node.js Server、Docker、Static Export 与适配器等形态，各自限制不同（https://nextjs.org/docs/app/getting-started/deploying）。

选 Vite 系（TanStack Router SPA / TanStack Start）时工具链统一，代价是没有 Next.js 的生态与内建约定。

### 认证层的选择差异

| 候选 | 定位 | 适合 |
|---|---|---|
| Better Auth | 框架无关的 TS 认证与授权框架，带插件生态 | 自建应用，要自己管数据库 |
| Auth.js | 完整的 Web 应用认证方案 | 已有 Next.js 生态，或需要标准 OAuth 流程 |
| Supabase Auth | Supabase 的托管认证服务 | 已经在用 Supabase 生态 |

⚠️ **Lucia 已废弃**：官方站写明 "Lucia was deprecated in March 2025"（https://lucia-auth.com/），官方给出的替代是单文件实现 `code/auth_session.ts`。不要在新选型里选它。

### 数据库引擎的选法

见 `database-engines.md`（TS 轨与 Python 轨共用）：候选与官方定位、SQLite 并发写入限制、MongoDB 驱动选择、引擎与 ORM/认证/部署的耦合。

要点：**引擎选择与 ORM 选择是两个决定** —— Prisma 与 Drizzle 都官方支持多种数据库。引擎对部署复杂度的影响最大：PostgreSQL / MySQL / MongoDB 是独立服务，部署层要包含它们；SQLite 是进程内文件，只需备份文件。
