# 术语表以 skill 内 references/glossary.md 为单一权威

全栈术语的条目（专业词、白话解释、为什么影响你、常见误用）存放在 `skills/fullstack-stack-guide/references/glossary.md`，是唯一权威来源。仓库根 `GLOSSARY.md` 只放本仓库自身的词汇（选型前提、技术硬约束、生态惯例、决策文件、变更影响评估、候选组合），并指向术语表。

原因：术语表要随技能分发。技能被 `npx skills add` 装到别的项目后，仓库根的 `GLOSSARY.md` 不会跟着走；术语条目放在仓库根会导致单独安装的技能失去术语能力。放在 skill 内则始终可用。

代价：仓库根 `GLOSSARY.md` 不再是「全量术语表」，探索本仓库的 agent 需要读 skill 内的文件才能拿到术语定义。
