# Full-Stack Stack Guide

本仓库是一个 **Agent Skill** 的家：`fullstack-stack-guide` —— 面向全栈项目（TypeScript 轨 / Python 轨）的**技术栈选型指导**技能。

它解决两个问题：

1. **选型**：一个全栈项目横跨十来个层（前端框架、后端、API 风格、数据库、ORM、认证、运行时、部署、测试）。哪些组合彼此不兼容、哪些只是社区惯例、版本号该钉在哪 —— 这些是**可查证的事实**，不必靠记忆或拍脑袋。
2. **术语**：用户抛需求时说的「前后端」「接口」「迁移」「部署」，和工程师说的不是一回事。技能在选型对话里同步把术语讲清楚，让用户知道自己参与决定了什么。

技能**不存版本事实**，只存**结构规则与查证路径**：不出现「当前版本是什么」的陈述，只写「怎么查官方文档、怎么判定兼容、怎么把证据落成文件」。版本必须在选型当时现查现证。详见 [ADR 0001](docs/adr/0001-no-version-snapshots.md)。

## 目录

```
skills/fullstack-stack-guide/
  SKILL.md                      技能主体：三阶段流程 + 边界 + 复核
  references/
    artifacts.md                产物格式：决策表、版本证据、变更影响评估
    constraints-python.md       Python 轨：层候选 + 技术硬约束 + 生态惯例
    constraints-typescript.md   TS 轨：层候选 + 技术硬约束 + 生态惯例
    database-engines.md         数据库引擎选型（两轨共用）
    glossary.md                 全栈术语条目（专业词 / 白话 / 为什么影响你 / 常见误用）
    version-verification.md     版本查证流程（npm / PyPI 两套字段）
    templates/                  可直接复制的产物模板
```

## 安装

```bash
# 全局，跨项目可用
npx skills add meisijiya/full-stack-skill -g

# 只装本项目
npx skills add meisijiya/full-stack-skill

# 先看有哪些 skill 可装
npx skills add meisijiya/full-stack-skill --list
```

`skills` CLI 会把技能装到各 agent 的技能目录（`.agents/skills/`、`.claude/skills/` 等）。手动安装等价物：

```bash
git clone https://github.com/meisijiya/full-stack-skill.git
cp -r full-stack-skill/skills/fullstack-stack-guide ~/.agents/skills/
```

## 使用

技能是 **model-invoked** 的：描述里带了触发场景，agent 在遇到选型、版本兼容、术语解释时会自己加载它。也可以手动调用：

```
/skill:fullstack-stack-guide
```

典型触发：

- 「我要做一个全栈项目，该用什么技术栈」
- 「这个组合能不能跑」
- 「Drizzle 和 Prisma 选哪个」
- 「需求变了，现在这套还合适吗」
- 「什么是 ORM / 什么是 RPC」

## 配置

**无需额外配置**，无密钥、无账号、无初始化步骤。安装完即可使用。

技能只在被触发时读文件、写文件，不需要常驻进程。

查证版本时需要本地有 `npm`（TS 轨）或 `curl` 与 `pip`（Python 轨）。这些是查证命令的依赖，不是技能的运行依赖——不查版本也能用技能，只是版本证据会缺失。

技能会在使用方项目里创建 `docs/stack/` 目录并可能修改该项目的 `AGENTS.md`（追加一段 stack 小节）。这是它唯一写入外部状态的动作。

## 使用后它会在你的项目里留下什么

```
docs/stack/
  decision.md          选型决策表（每层选型 + 理由 + 代价）
  version-evidence.md  版本查证记录（命令 + 输出 + 日期）
  change-impact.md     变更影响评估（开发中复核时产出）
```

并在项目 `AGENTS.md` 追加一段指向 `docs/stack/decision.md` 的小节，让后续会话能读到选型结论。

## 适用边界

**技能负责**：技术栈选型、候选比较、组合兼容性判定、版本兼容查证、术语解释、开发中因需求变更而评估「继续 / 降级 / 换栈」。

**技能不负责**：需求澄清与需求对齐、架构设计、代码编写、代码审查、部署运维操作。这些交给对应技能或你自己决定。

**不在选型范围内的**：技能给的是通用技术栈指导，不替你做具体产品的方案设计。用户已有的、已验证的技术栈选择不需要它来翻案——除非需求变了触及了选型前提。

## 数据与权限

- 技能**不收集、不上传任何数据**。没有遥测、没有网络上报、没有密钥存储。
- 唯一出站请求是查证版本时的 `npm view` / `curl https://pypi.org/...`，都是公开只读请求。
- 唯一写入外部状态的动作：在使用方项目创建 `docs/stack/` 目录下的文件，以及在该项目 `AGENTS.md` 追加一段 stack 小节。写入前会先检查现有内容。

## 兼容性

| 维度 | 状态 |
|---|---|
| 操作系统 | 无平台限定。技能是文档与流程，macOS / Windows / Linux 均可。查证命令按各平台通用写法给出 |
| 宿主 | 任何支持 Agent Skills 目录约定（`SKILL.md` + frontmatter）的宿主。已验证 `npx skills add` 安装后被识别 |
| 运行时 | 无运行时依赖。技能不含脚本 |
| 查证工具 | TS 轨需 `npm`；Python 轨需 `curl` 与 `pip`（可选 `jq`）。缺失时技能仍可用，但版本证据不完整 |
| 网络 | 查证版本需要访问 npm registry 与 PyPI。离线环境下技能仍可做术语解释与约束核对，版本需人工提供 |
| 无子 Agent 能力 | 支持。技能是给单个 Agent 读的流程与参考资料，不需要委派 |

## 设计决策

见 `docs/adr/`。其中影响使用体验的三条：

- [0001](docs/adr/0001-no-version-snapshots.md) 不存版本事实，只存结构规则与查证路径——技能不跟踪版本演进，避免变成需要持续跟进维护的版本表
- [0002](docs/adr/0002-constraint-layers.md) 约束分「技术硬约束」与「生态惯例」两层
- [0003](docs/adr/0003-scope-boundary.md) 只做选型与术语，不做需求对齐

## 许可

MIT，见 [LICENSE](LICENSE)。
