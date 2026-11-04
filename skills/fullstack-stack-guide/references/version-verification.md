# 版本查证流程

本技能**不存版本号**。每次选型都要现查现证，把命令与输出写进 `docs/stack/version-evidence.md`。

## 为什么要查

- npm 上 `latest` 标签**不等于稳定版**。实测 2026-11-04：`prisma` 的 latest 指向 `8.0.0-rc.30`（RC），`drizzle-orm` 的 latest 停在 `0.45.4` 而 `1.0.0-rc.5` 已发布。照抄 `npm i pkg@latest` 会拿到预发布版。
- 文档推荐版本 ≠ 实际安装版本。实测：`@orpc/server` 的 npm latest 是 1.16.1（v1 稳定），但 oRPC 官网文档明确 "These docs cover oRPC v2, currently in beta"，安装命令是 `npm install @orpc/server@beta`。
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

`npm view` 官方文档：https://docs.npmjs.com/cli/v11/commands/npm-view/ 。字段访问示例 `npm view express "dist-tags.latest"`。

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
3. 目标包的 `peerDependencies` 与已选其它包的**实际版本**逐一比对：对方版本不在你的 peer 范围内 → 硬冲突，必须换版本或换包。
4. `peerDependenciesMeta` 里标 `optional: true` 的 peer 缺失不算冲突。
5. 框架与工具链的耦合要看官方 changelog，不能只看 peer 范围——例如工具链精确锁定 linter / formatter / test runner 版本时，升 linter 只能升工具链本身。

### 交叉验证

peer 范围只说明「声明上兼容」。**真正的兼容要读官方 changelog / migration guide**：有破坏性变更的版本区间，peer 范围不会体现。查 `<pkg> repository.url` 找到源码仓库，翻 releases 或 migration guide。

## PyPI 查证

### PyPI JSON API 的真实字段

⚠️ **`https://pypi.org/pypi/<pkg>/json` 没有 `dist-tags` 键。** 实测顶层键只有：`info`、`last_serial`、`ownership`、`releases`、`urls`、`vulnerabilities`。官方 JSON API 文档同样不含该键（https://docs.pypi.org/api/json/）。

| 要找的 | 字段 |
|---|---|
| 当前版本 | `info.version` |
| Python 版本要求 | `info.requires_python`（如 `">=3.9"`） |
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
3. `info.requires_dist` 里的上限约束（如 `pydantic<3.0.0`）→ 与已选其它包版本冲突则必须换版本。
4. `releases` 里被 `yanked: true` 的版本不选。
5. **维护状态要单独查**：`info.version` 只说明最新版号，不说明项目是否还活着。看仓库最近提交与 issue 响应。已知：arq 官方标注 "maintenance only mode"，Taskiq 分类器标 Alpha，Motor 已弃用。
6. 破坏性变更读官方 migration guide：Pydantic v1→v2、SQLAlchemy 1.4→2.0 都有官方迁移指南，列明被移除的 API。

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

## 已知坑（实测于 2026-11-04，形态会漂移，用前复核）

- **预发布版占 latest**：`prisma` latest 指向 RC；`drizzle-orm` latest 停在 0.x 而 1.0 RC 已发布。查 dist-tags 才能看出。
- **文档版本 ≠ npm 版本**：oRPC 官网文档讲 v2（beta），npm latest 是 v1 稳定。读完文档要去查对应 tag。
- **Prisma v7 起 `migrate dev` 不再自动跑 `prisma generate`**（v6 会自动跑）。跨大版本时这会静默打断流程。
- **生成代码先于类型检查**：TanStack Router 的路由树由构建生成，新工程先构建再类型检查。
- **工具链内部版本锁**：vite-plus 精确锁定 oxlint / oxfmt / vitest 版本，升这些只能升 vite-plus 本身。
- **Windows 上 pnpm 原生二进制缺失**：`packageManager` 钉了新 pnpm 版本时 shim 可能报「不是内部或外部命令」，需在该版本目录跑一次 `node install.js` 补原生二进制。
