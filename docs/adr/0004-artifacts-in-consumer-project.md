# 选型产物落在使用方项目的 docs/stack/

技能被使用后，产物写进**使用方项目**的 `docs/stack/`，并在该项目的 `AGENTS.md` 增加一段 stack 小节指向它。技能自身仓库不存放任何具体项目的选型结果。

目录结构：

```
docs/stack/
  decision.md            选型决策表
  version-evidence.md    版本查证记录（命令 + 输出 + 日期）
  change-impact.md       变更影响评估（开发中复核产出）
```

原因：选型结论的价值随项目生命周期存在。落在使用方项目里，后续任何会话都能读到它，需求变更时才有依据判断「继续 / 降级 / 换栈」。只留在对话里，下一个会话就失忆。

`AGENTS.md` 小节遵循本仓库既有的 agent 文档约定：短摘要 + 指针，指向 `docs/stack/decision.md`。

代价：使用方项目需要接受 `docs/stack/` 这个约定。技能在写入前先检查该项目的 `AGENTS.md` 是否已有 stack 小节，避免重复添加。
