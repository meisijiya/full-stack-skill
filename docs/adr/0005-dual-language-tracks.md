# TypeScript 与 Python 双轨平级覆盖

技能同时覆盖 TypeScript 与 Python 两条全栈轨，两轨的层结构平级（层候选表 + 硬约束 + 生态惯例 + 版本查证流程），而不是把 Python 当成 TS 的附属说明。

原因：两条轨的技术现实差异是结构性的，不对称覆盖会逼技能给出误导建议。差异包括：TS 轨存在「框架自带 server」这一层（Next.js / TanStack Start），Python 轨没有对应物（ASGI 应用与框架分离）；TS 轨的 ORM 与数据库约束宽松（Prisma 与 Drizzle 均官方支持多种数据库），Python 轨的 ORM 与迁移工具强绑定（Alembic 绑定 SQLAlchemy，Django 自带 makemigrations）；TS 轨有 `uv` 式的统一 CLI（Vite+ 的 `vp`），Python 轨有 `uv` 但职责不同。

代价：维护面翻倍。版本查证流程可共用（npm 与 PyPI 两套字段需分别说明），约束与候选表必须分别维护。
