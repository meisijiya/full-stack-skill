# 版本查证流程

## 版本知识的使用边界

本技能**不保存任何版本号快照**，也不跟踪版本演进。它只保存「怎么查」和「怎么判断」。任何写在这里的版本相关表述都是**结构规则**（例如「大版本换代时常有破坏性变更」「latest 标签可能指向预发布」），不是当前版本事实。选型时一律现查。

每次选型都要现查现证，把命令与输出写进 `docs/stack/version-evidence.md`。

## 为什么要查

- npm 上 `latest` 标签**不等于稳定版**：它可能指向预发布版本，也可能因为该包还没发正式版而长期停在早期大版本。**当场判断**：`npm view <pkg> dist-tags --json`——值里含 `-beta` / `-rc` / `-alpha` / `-canary` / `-next` / `-preview` 即为预发布。照抄 `npm i pkg@latest` 会拿到预发布版。
- 文档推荐版本 ≠ 实际安装版本：官方文档站可能整站覆盖一个尚未发布的版本线（页面顶部会有一句 "These docs cover …, currently in beta" 这类声明），而 npm 上 `latest` 指向的是另一条线。**当场判断**：先读文档站的版本声明，再 `npm view <pkg> dist-tags` 对照；文档要求装预发布就用对应 tag 装（如 `npm install <pkg>@beta`），别默认 `latest`。
- 版本下限藏在 `engines` 和 `peerDependencies` 里，不看清单就会撞上。

## npm 查证

### 命令

```bash
# 1. 看所有 dist-tags（latest 之外还有什么在跑）
npm view <pkg> dist-tags --json

# 2. 判断 latest 是否预发布：值里含 -beta / -rc / -alpha / -canary / -next / -preview 即为预发布
npm view <pkg> dist-tags.latest

# 3. 运行时下限
npm view <pkg> engines --json

# 4. peer 依赖范围（决定与别的包能否共存）
npm view <pkg> peerDependencies --json

# 5. 可选 peer 是否强制
npm view <pkg> peerDependenciesMeta --json

# 6. 全部可用版本
npm view <pkg> versions --json

# 7. 官方仓库地址（找 changelog 用）
npm view <pkg> repository.url
```

`npm view` 官方文档：https://docs.npmjs.com/cli/commands/npm-view/ （该路径会自动跳到当前 CLI 大版本；想固定读哪一版，路径里带 `v<大版本>`）。字段访问示例 `npm view express "dist-tags.latest"`。

### registry API

```bash
# 完整文档：顶层键 _id / _rev / name / dist-tags / versions
curl -s https://registry.npmjs.org/<pkg> | jq '.["dist-tags"], (.versions | keys | .[-5:])'

# 单版本 manifest：含 engines / peerDependencies / peerDependenciesMeta / dependencies
curl -s https://registry.npmjs.org/<pkg>/latest | jq '{version, engines, peerDependencies, peerDependenciesMeta}'
```

### 判定规则

1. `dist-tags.latest` 含预发布标识 → **不能用 latest**，改用具体稳定版本号并说明理由。
2. 目标包的 `engines.node` 范围 → 决定 Node 版本下限。Node 版本还要对照官方 LTS 表（https://nodejs.org/en/about/previous-releases）。
3. **多个包的 Node 范围取交集。** 整个项目能跑的 Node 版本，是所有依赖的 `engines.node` 范围的交集，不是主框架一个包的。测试运行器、构建工具的门槛常比主框架高。
4. 目标包的 `peerDependencies` 与已选其它包的**实际版本**逐一比对：对方版本不在你的 peer 范围内 → 硬冲突，必须换版本或换包。
5. `peerDependenciesMeta` 里标 `optional: true` 的 peer 缺失不算冲突。
6. 框架与工具链的耦合要看官方 changelog，不能只看 peer 范围——例如工具链精确锁定 linter / formatter / test runner 版本时，升 linter 只能升工具链本身。

### 包没有声明 `engines` 时的判定

`engines` 在 package.json 里是**可选**字段——npm 官方说明未指定时不会阻止安装（https://docs.npmjs.com/cli/configuring-npm/package-json#engines ，该路径会自动跳到当前 CLI 大版本）。所以「查不到 engines」不等于「无版本下限」，按这个顺序找：

1. 包自身 README 里的 Requirements / Supported Node.js version 段落
2. 包的官方文档站或支持政策页
3. 该包的官方 CI 配置里用的 Node 版本
4. 都没有 → **在决策表里标「未查得 Node 下限官方声明」**，并按项目要跑的最老 Node 版本保守选型

已知的实际例子：`@orpc/server` 与 `better-auth` 的 npm 清单无 `engines` 字段；`@orpc/server` 另有官方 Runtime Requirements 页（https://orpc.dev/docs/requirements），`better-auth` 则需回落到第 1–3 条。

### 查证 TypeScript 大版本兼容性

TypeScript 正处于大版本换代期，换代会影响两件事：**编译器 API 的形态**（有些大版本不带 compiler API，或 API 另行单独发布）与**各库声明的 TS 测试下限**（每个库各自声明，不统一）。

查证路径：

1. 官方发布说明写明换代带来的兼容影响（编译器重写、API 形态变化等）：https://devblogs.microsoft.com/typescript/
2. 某库声明的 TS 测试下限：查该库文档的 Requirements 章节（例：Zod 的 Requirements 节 https://zod.dev/ ）——**这是该库的下限，不是 TypeScript 的下限**。
3. 依赖 compiler API 的工具链（typescript-eslint 等）当前支持哪些 TS 版本：查它的支持矩阵（https://typescript-eslint.io/users/dependency-versions/ ）。

判定规则：

- 查 `<包>` 官方支持矩阵里是否列出你打算用的 TS 大版本 → 列出则可放心用
- 未列出 → **标「未查得官方声明」**，并查该包最近版本的 changelog 是否有相关修复
- 依赖 TS compiler API 的工具链在换代版本下 → 一律标「未查得官方声明」并给出替代（并排装官方说明里的旧版编译器包，具体包名查该工具的官方文档）

### 非 npm 分发的东西怎么查

有些候选不是 npm 包，没有 `npm view` 可查：

- **shadcn/ui** —— 组件代码复制进项目仓库，不是 npm 依赖。查证对象变成它的上游（Base UI / Radix）与 CLI 版本，查 `<上游包>` 的 npm 清单即可。决策表里这一层记「组件库，无独立版本号，随上游组件更新」。
- **自建/框架内置的能力** —— 如 Django 的 ORM 与认证、Django 的 `makemigrations`。查证对象是框架本身的版本与其官方文档对应章节。
- **Docker 镜像** —— 查镜像仓库的 tag 列表与 digest，钉具体 tag 或 digest，不要用 `latest`。

### 交叉验证

peer 范围只说明「声明上兼容」。**真正的兼容要读官方 changelog / migration guide**：有破坏性变更的版本区间，peer 范围不会体现。查 `<pkg> repository.url` 找到源码仓库，翻 releases 或 migration guide。

## PyPI 查证

### PyPI JSON API 的真实字段

⚠️ **`https://pypi.org/pypi/<pkg>/json` 没有 `dist-tags` 键。** 实测顶层键只有：`info`、`last_serial`、`ownership`、`releases`、`urls`、`vulnerabilities`。官方 JSON API 文档同样不含该键（https://docs.pypi.org/api/json/）。

| 要找的 | 字段 |
|---|---|
| 当前版本 | `info.version` |
| Python 版本要求 | `info.requires_python`（形如 `">=3.9"`，**此为样式示例，实际下限现查**） |
| 是否被撤回 | `info.yanked`、`info.yanked_reason` |
| 全部历史版本 | `releases` —— 按版本号字符串键映射到文件数组 |
| 单个文件是否撤回 | `releases.<ver>[].yanked`、`yanked_reason` |
| 当前版本的文件列表 | `urls` —— 与 `releases.<info.version>` 同构 |

文件对象字段：`filename, url, upload_time, upload_time_iso_8601, requires_python, yanked, yanked_reason, digests{sha256,md5,blake2b_256}, packagetype, python_version, size, comment_text, has_sig`。

### 命令

```bash
# 当前版本与 Python 要求
curl -s https://pypi.org/pypi/<pkg>/json | jq '{version: .info.version, requires_python: .info.requires_python, yanked: .info.yanked}'

# 可用版本列表（pip 特有；uv 无对应子命令）
pip index versions <pkg>

# 依赖关系
curl -s https://pypi.org/pypi/<pkg>/json | jq '.info.requires_dist'

# 是否预发布：版本串含 dev / rc / a / b
curl -s https://pypi.org/pypi/<pkg>/json | jq '.info.version'
```

`uv pip` 的官方子命令是 `compile, sync, install, uninstall, freeze, list, show, tree, check`——**没有 `index`**，所以 `uv pip index versions` 不存在（https://docs.astral.sh/uv/reference/cli/）。用 `pip index versions`。

### 判定规则

1. `info.version` 含 `dev` / `rc` / `a` / `b` 后缀 → 预发布，不直接用。
2. `info.requires_python` 与目标 Python 版本比对 → 不满足则该版本不可用。
3. `info.requires_dist` 里的上限约束（形如 `pydantic<3.0.0`）→ 与已选其它包版本冲突则必须换版本。**这是约束样式的示例，不是当前上限**——每次以该包当前发布清单为准。
4. `releases` 里被 `yanked: true` 的版本不选。
5. **维护状态要单独查**：`info.version` 只说明最新版号，不说明项目是否还活着。看仓库最近提交与 issue 响应。已知：arq 官方标注 "maintenance only mode"，Taskiq 分类器标 Alpha，Motor 已弃用。
6. 破坏性变更读官方 migration guide：主流库在跨大版本时都提供官方迁移指南（列明被移除的 API）。**具体哪些大版本之间有迁移指南，查该包官方文档的 migration / upgrade 章节**，不要凭记忆。

## 两轨通用步骤

选型定下候选组合后，逐层执行：

1. 对每个包跑上面的查证命令。
2. 把**命令原文**与**输出原文**写进 `docs/stack/version-evidence.md`，附日期。
3. 把冲突逐条消解：换版本、换包、或标注「未查得官方兼容声明」。
4. 查不到官方声明的层，在决策表里标「未查得官方声明」——**不要**用「应该兼容」代替。

### 记录格式

```markdown
### <层> · <包名>

- 日期：YYYY-MM-DD
- 命令：
  ```
  npm view <pkg> dist-tags --json
  ```
- 输出：
  ```json
  { ... }
  ```
- 结论：<版本号> / 冲突与消解 / 未查得官方声明
```

## 选型落地后的最小验证顺序

版本查证只能证明「声明上兼容」。**证明组合真能跑**要靠最小切片验证：选型定下、动手写业务前，按这个顺序走一遍，每步都是下一步的前置。

1. **装依赖** —— 锁死决策表里的确切版本（不用范围）
2. **构建** —— 证明路由代码生成与打包链路通。TS 轨注意 TanStack Router 的路由树由构建生成，**必须先构建再类型检查**
3. **类型检查与 lint** —— 证明静态检查链路通
4. **迁移生成与执行** —— 证明数据层链路通。TS 轨 Drizzle 先 `generate` 再 `migrate`；Python 轨先 `alembic revision --autogenerate` 再 `upgrade`。注意**跨 Prisma 大版本时 `migrate dev` 是否自动触发 `generate` 有过变化**，按官方当前文档执行（https://www.prisma.io/docs/cli/migrate/dev ）
5. **测试** —— 证明测试链路通
6. **浏览器/接口手动过一条主流程** —— 证明应用真能起

任一步失败，**先怀疑组合本身**（回看硬约束与 peer 范围），不要改业务代码。

验证结果追加到 `docs/stack/version-evidence.md` 末尾，作为组合可行性的证据。

### Vite+ 工具链的具体纪律

若选了 Vite+（`vp` 统一 CLI），vite 是通过**别名**指向 vite-plus 的核心包，且别名版本与 vite-plus 版本必须一致——不同步会报版本解析失败（如 `Failed to resolve vite command: Expected @voidzero-dev/vite-plus-core@X, but found ...@Y`）。项目里通常是 `pnpm-workspace.yaml` 的 catalog 里两处同改，或跑 `vp migrate` 对齐。

vite-plus 精确锁定 oxlint / oxfmt / vitest 版本，**升这些只能升 vite-plus 本身**。lint 规则报错 "Rule '...' not found in plugin" 通常是 oxlint 版本与规则集不匹配，升 vite-plus 即可（具体形态会漂移，查 `viteplus.dev/guide`）。

## 已知坑（形态会漂移，用前复核）

- **预发布版占 latest**：某些包的 `latest` 指向 RC，甚至长期停在早期大版本而正式版还没发。**当场判断**：`npm view <pkg> dist-tags --json`，看有没有独立的稳定 tag。
- **文档版本 ≠ npm 版本**：官网文档可能整站覆盖一个尚未发布的版本线，npm `latest` 却是另一条线。读完文档去查对应 tag，别默认 `latest`。
- **跨 Prisma 大版本时 `migrate dev` 是否自动跑 `prisma generate` 有过变化**：这会静默打断流程，按官方当前文档执行（https://www.prisma.io/docs/cli/migrate/dev ）。
- **生成代码先于类型检查**：TanStack Router 的路由树由构建生成，新工程先构建再类型检查。
- **工具链内部版本锁**：vite-plus 精确锁定 oxlint / oxfmt / vitest 版本，升这些只能升 vite-plus 本身。
- **Windows 上 pnpm 原生二进制缺失**：`packageManager` 钉了新 pnpm 版本时 shim 可能报「不是内部或外部命令」，需在该版本目录跑一次 `node install.js` 补原生二进制。
