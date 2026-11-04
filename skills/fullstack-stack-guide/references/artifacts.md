# 产物格式

产物写进**使用方项目**的 `docs/stack/`。技能自身仓库不存放任何具体项目的选型结果。

技能自身仓库（`meisijiya/full-stack-skill`）的根 `docs/` 只放 agent 文档与 ADR，不是产物目录。在本仓库里做验证或演练时，把产物写到仓库外的临时目录，不要混进 `docs/`。

```
docs/stack/
  decision.md          选型决策表
  version-evidence.md  版本查证记录
  change-impact.md     变更影响评估（开发中复核时新建）
```

模板在 `references/templates/`，可直接复制。

## decision.md · 选型决策表

结构：

1. **选型前提** —— 阶段一收集的硬事实，逐条列出。这是决策的依据，也是日后判断「前提是否变了」的基准。
2. **候选组合与选择理由** —— 曾经考虑哪几套，为什么选了这套。
3. **每层决策表** —— 层、选型、版本、理由、代价。
4. **关键取舍** —— 这套方案放弃了什么，哪天可能会后悔。
5. **未查得官方声明的层** —— 明确列出，不要藏。
6. **复核条件** —— 什么情况下该重新评估。

模板见 `templates/decision.md`。

## version-evidence.md · 版本查证记录

按层分节，每节记：命令原文、输出原文、日期、结论。

```markdown
### 数据层 · <包名>

- 日期：YYYY-MM-DD
- 命令：
  ```
  npm view <pkg> dist-tags --json
  ```
- 输出：
  ```json
  { "latest": "<版本号>", "next": "<预发布版本号>" }
  ```
- 结论：<版本号> / <为什么不追预发布> / 未查得官方声明
```

模板见 `templates/version-evidence.md`。

## change-impact.md · 变更影响评估

需求变更时新建（**不覆盖**历史，一次变更一节追加）：

1. **变更内容** —— 需求是什么。
2. **触及的层** —— 读 `decision.md` 找出受影响的层。
3. **三个选项与代价** —— 继续 / 降级 / 换栈，每个写清代价。
4. **结论** —— 用户选了什么，日期。

模板见 `templates/change-impact.md`。

## 使用方项目的 AGENTS.md 小节

技能在用户选定后，检查使用方项目的 `AGENTS.md`：

- 已有 stack 小节 → 只更新指向的文件路径，不重复添加。
- 没有 `AGENTS.md` → **不新建**。改在 `docs/stack/decision.md` 里加一行指向自身的说明，并在回复里告知用户：若希望后续会话自动读到选型结论，可在项目 `AGENTS.md` 加一段指向 `docs/stack/decision.md` 的小节。是否新建由用户决定。
- 有 `AGENTS.md` 但结构不同 → 跟随该项目既有的小节风格，不自造格式。

小节内容要短：一句说明 + 指向 `docs/stack/decision.md` 的指针。

```markdown
## Stack

技术栈选型决策见 [docs/stack/decision.md](docs/stack/decision.md)。
需求变更可能影响选型时，先读该文件再评估「继续 / 降级 / 换栈」。
版本查证记录见 [docs/stack/version-evidence.md](docs/stack/version-evidence.md)。
```

不要在 AGENTS.md 里复制决策内容——单一权威在 `docs/stack/decision.md`。
