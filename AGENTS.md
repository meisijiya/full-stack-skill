# AGENTS.md

## Agent skills

### Issue tracker

Issues 存放在本仓库的 GitHub Issues（`meisijiya/full-stack-skill`），通过 `gh` CLI 读写。See `docs/agents/issue-tracker.md`.

### Triage labels

五个规范分类角色：标签字符串与角色名相同（`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`）。See `docs/agents/triage-labels.md`.

### Domain docs

单上下文布局：根目录 `GLOSSARY.md` + `docs/adr/`。See `docs/agents/domain.md`.

## 本仓库提供的 skill

`fullstack-stack-guide` —— 全栈技术栈选型指导（TypeScript 轨 / Python 轨）。见 `skills/fullstack-stack-guide/SKILL.md`。

选型与术语解释由它承担；需求对齐、架构设计、代码审查不归它管。
