# Full-Stack Stack Guide

本仓库是一个 **Agent Skill** 的家：`fullstack-stack-guide` —— 面向全栈项目（TypeScript 轨 / Python 轨）的**技术栈选型指导**技能。

它解决两个问题：

1. **选型**：一个全栈项目横跨十来个层（前端框架、后端、API 风格、数据库、ORM、认证、运行时、部署、测试）。哪些组合彼此不兼容、哪些只是社区惯例、版本号该钉在哪 —— 这些是**可查证的事实**，不必靠记忆或拍脑袋。
2. **术语**：用户抛需求时说的「前后端」「接口」「迁移」「部署」，和工程师说的不是一回事。技能在选型对话里同步把术语讲清楚，让用户知道自己参与决定了什么。

技能不存版本号。它存的是**查证流程**：怎么查官方文档、怎么判定兼容、怎么把证据落成文件。版本号必须在选型当时现查现证。

## 目录

```
skills/fullstack-stack-guide/
  SKILL.md                      技能主体：三阶段流程 + 边界 + 复核
  references/
    glossary.md                 全栈术语条目（专业词 / 白话 / 为什么影响你 / 常见误用）
    constraints-typescript.md   TS 轨：层候选 + 技术硬约束 + 生态惯例
    constraints-python.md       Python 轨：层候选 + 技术硬约束 + 生态惯例
    version-verification.md     版本查证流程（npm / PyPI 两套字段）
    artifacts.md                产物格式：决策表、版本证据、变更影响评估
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

## 使用后它会在你的项目里留下什么

```
docs/stack/
  decision.md          选型决策表（每层选型 + 理由 + 代价）
  version-evidence.md  版本查证记录（命令 + 输出 + 日期）
  change-impact.md     变更影响评估（开发中复核时产出）
```

并在项目 `AGENTS.md` 追加一段指向 `docs/stack/decision.md` 的小节，让后续会话能读到选型结论。

## 设计决策

见 `docs/adr/`。其中影响使用体验的三条：

- [0001](docs/adr/0001-no-version-snapshots.md) 不存版本号，只存查证流程
- [0002](docs/adr/0002-constraint-layers.md) 约束分「技术硬约束」与「生态惯例」两层
- [0003](docs/adr/0003-scope-boundary.md) 只做选型与术语，不做需求对齐

## 许可

MIT，见 [LICENSE](LICENSE)。
