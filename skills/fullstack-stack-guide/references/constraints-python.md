# Python 轨：层候选、硬约束、生态惯例

约束同样分两层：**技术硬约束**（官方文档可证实）与**生态惯例**（常见搭配，非强制）。

## 层候选

| 层 | 候选与官方定位 |
|---|---|
| Web 框架 | **FastAPI** — "a modern, fast (high-performance), web framework for building APIs with Python based on standard Python type hints."（https://fastapi.tiangolo.com/）<br>**Django** — "a high-level Python web framework that encourages rapid development and clean, pragmatic design."（https://www.djangoproject.com/）<br>**Flask** — "a lightweight WSGI web application framework."（https://flask.palletsprojects.com/en/stable/）<br>**Litestar** — "The Modern Python ASGI Framework Built to scale"；"Fast APIs with automatic OpenAPI documentation, dependency injection, type safety"（https://litestar.dev/）<br>**Sanic** — "The lightning-fast asynchronous Python web framework"（官方自述的 Python 下限随其发布变化，选型时查 Sanic 安装文档的 Requirements 节：https://sanic.dev/en/guide/getting-started.html ） |
| 数据层 · ORM | **SQLAlchemy 2.x** — 从 2.0 起提供 async 支持（`sqlalchemy.ext.asyncio`，https://docs.sqlalchemy.org/en/latest/orm/extensions/asyncio.html）；「2.x」是 API 世代标记，不是现值<br>**Django ORM** — Django 内置<br>**Tortoise ORM** — "an easy-to-use asyncio ORM inspired by Django"；"a lightweight, async-native Object-Relational Mapper"（支持的 Python 范围随其发布变化，选型时查官方安装文档与 PyPI 的 requires_python：https://tortoise.github.io/ ）<br>**SQLModel** — "based on Python type annotations, and powered by Pydantic and SQLAlchemy"（https://sqlmodel.tiangolo.com/）<br>**peewee** — "a simple and small ORM"；"Peewee's async extension provides asyncio-compatible database backends built on the standard async drivers, aiosqlite, asyncpg and aiomysql."（http://docs.peewee-orm.com/en/latest/peewee/asyncio.html） |
| 数据层 · 数据库引擎 | 见 `database-engines.md`（TS 轨与 Python 轨共用） |
| 迁移 | **Alembic** — "a lightweight database migration tool for usage with the SQLAlchemy Database Toolkit for Python."（https://alembic.sqlalchemy.org/）<br>**Django makemigrations / migrate** — Django 内置 |
| 校验 | **Pydantic v2** — "Pydantic is the most widely used data validation library for Python."（https://docs.pydantic.dev/latest/）<br>**attrs** — "the Python package that will bring back the joy of writing classes by relieving you from the drudgery of implementing object protocols"（https://www.attrs.org/）<br>**cattrs** — "cattrs is a Swiss Army knife for (un)structuring and validating data in Python."（https://catt.rs/en/stable/）<br>**msgspec** — "A fast serialization and validation library, with builtin support for JSON, MessagePack, YAML, and TOML."（https://pypi.org/project/msgspec/） |
| 认证 | **Django 内置认证** — "Django comes with a user authentication system. It handles user accounts, groups, permissions and cookie-based user sessions."（https://docs.djangoproject.com/en/stable/topics/auth/）<br>**Authlib** — "The ultimate Python library in building OAuth and OpenID Connect servers and clients."（https://docs.authlib.org/）<br>**fastapi-users** — "Add quickly a registration and authentication system to your FastAPI project."（https://fastapi-users.github.io/fastapi-users/） |
| 依赖管理 | **uv** — "An extremely fast Python package and project manager, written in Rust."；"A single tool to replace pip, pip-tools, pipx, poetry, pyenv, twine, virtualenv, and more."（https://docs.astral.sh/uv/）<br>**Poetry** — "Python packaging and dependency management made easy"（https://python-poetry.org/） |
| 测试 | **pytest** — "The pytest framework makes it easy to write small tests, yet scales to support complex functional testing"（https://pypi.org/project/pytest/）<br>**pytest-asyncio** — "a pytest plugin… facilitates testing of code that uses the asyncio library"（https://pypi.org/project/pytest-asyncio/）<br>**httpx** — "a fully featured HTTP client library for Python 3… provides both sync and async APIs"（https://pypi.org/project/httpx/） |
| 运行时与服务器 | **ASGI**（uvicorn / granian）、**WSGI/ASGI**（gunicorn 原生支持两者）—— 见硬约束节<br>**Uvicorn** — "an ASGI web server implementation for Python."（https://uvicorn.dev/）<br>**Gunicorn** — "Gunicorn includes a native ASGI worker that enables running async Python web frameworks like FastAPI, Starlette, and Quart without external dependencies like Uvicorn."（https://gunicorn.org/asgi/） |
| 后台任务 | **Celery** — "Distributed Task Queue."（https://pypi.org/project/celery/）<br>**Dramatiq** — "A fast and reliable distributed task processing library for Python 3."（https://pypi.org/project/dramatiq/）<br>**Taskiq** — "Distributed task queue with full async support"；分类器标注 `Development Status :: 3 - Alpha`（https://pypi.org/project/taskiq/）<br>**arq** — "Job queues in python with asyncio and redis"；官方标注 **"arq is in maintenance only mode"**（https://pypi.org/project/arq/） |
| 部署 · 打包 | **Docker** — 官方 Python 镜像："The python images come in many flavors, each designed for a specific use case."；`python:<version>` 是默认镜像，另有 `-slim` / `-alpine`（https://github.com/docker-library/docs/blob/master/python/README.md） |
| 部署 · 进程与反向代理 | **Nginx** — "nginx ("engine x") is an HTTP web server, reverse proxy, content cache, load balancer, TCP/UDP proxy server, and mail proxy server."（https://nginx.org/en/）<br>**Caddy** — "a powerful, enterprise-ready, open source web server with automatic HTTPS written in Go"（https://caddyserver.com/） |

## 技术硬约束

### FastAPI 的版本策略要求钉版本

官方在其版本说明页明确：每个版本都可能包含破坏性变更（遵循语义化版本约定），因此**要求用户钉住 `fastapi` 版本**（https://fastapi.tiangolo.com/deployment/versions/）。

**后果**：FastAPI 不能用宽松版本范围，必须钉死具体版本或窄范围。这条规则与当前版本号无关——去该页看它对自己版本策略的现行表述。

### FastAPI 与 Pydantic 的绑定

官方："Pydantic includes the tests for **FastAPI** with its own tests, so new versions of Pydantic (above `1.0.0`) are always compatible with FastAPI. You can pin `Pydantic` to any version above `1.0.0` that works for you."（https://fastapi.tiangolo.com/deployment/versions/）（具体钉范围的示例区间随时间变化，以该页当前内容为准）。

**后果**：FastAPI 项目里 Pydantic 版本是安全的，但仍需钉范围——因为 Pydantic v2 自身对 v1 有破坏性变更。

### Pydantic v1 → v2 的破坏性变更

官方迁移指南（https://docs.pydantic.dev/latest/migration）列出的重命名：`BaseModel` 方法改名为 `model_*`、`construct()` → `model_construct()`、`dict()` → `model_dump()`、`json()` → `model_dump_json()`、`parse_obj()` → `model_validate()`；`json_encoders` 不再推荐；`GetterDict` 与部分解析 API 行为变更。

保留旧 API 的方式：`from pydantic.v1 import ...`。

**后果**：跨 v1/v2 的项目升级时，这批 API 是主要破坏点。

### SQLAlchemy 跨大版本迁移的破坏性变更

官方每个大版本都有对应的 migration guide，路径模式是 `/en/latest/changelog/migration_<跨版本号>.html`。官方迁移指南（https://docs.sqlalchemy.org/en/latest/changelog/migration_20.html）列出的 "major APIs that are no longer present"：`Bound MetaData and connectionless execution`、`Emulated autocommit on Connection`、`The Session.autocommit parameter / mode`、`List / keyword arguments to select()`、`Python 2 support`。

另含：autocommit 从 Core 与 ORM 移除；ORM Query 与 Core Select 合并；`Query.get()` 移到 Session；关系加载改用属性而非字符串；`in_()` / `not_in()` 不再接受显式子查询；Result 行表现为 named tuple。

**后果**：老项目升 SQLAlchemy 大版本必须先开 `RemovedIn20Warnings` 逐步清理（官方迁移步骤二、三步就是干这个）。

### Django 的 Python 版本矩阵

官方 FAQ 给出逐版本支持表（https://docs.djangoproject.com/en/stable/faq/install/）：Django 的 Python 版本支持是一张逐版本对照表，且随每个 Django 小版本更新（新增支持会写在对应小版本的 release notes 里）。选 Python 版本前查官方 FAQ 的 "What Python version can I use with Django?" 表。结论：**不要凭印象配版本，以该表当次内容为准。** 官方同时建议："Since newer versions of Python are often faster, have more features, and are better supported, the latest version of Python 3 is recommended."，且仅支持每个系列最新的微版本。

**后果**：Django 项目选 Python 版本要先查这张表，不能凭印象。

### Django 的主部署平台是 WSGI

Django 官方："Django's primary deployment platform is WSGI, the Python standard for web servers and applications."（https://docs.djangoproject.com/en/stable/howto/deployment/wsgi/）
同时官方也支持 ASGI："As well as WSGI, Django also supports deploying on ASGI, the emerging Python standard for asynchronous web servers and applications."（https://docs.djangoproject.com/en/stable/howto/deployment/asgi/）
**后果**：Django 默认按同步模型工作；走 ASGI 时要自己保证异步安全，官方 ASGI 页对此有说明。**不是「Django 不能异步」，而是「异步不是默认路径」**——这个区别会影响你选不选 Django。

### SQLite 的并发写入

见 `database-engines.md`。

### MongoDB 的 Python 驱动与 ODM

见 `database-engines.md`。

### ASGI 与 WSGI 的分野

Flask 官方自述是 WSGI 框架；Litestar 与 Sanic 官方自述为 ASGI 框架。Gunicorn 现在内置原生 ASGI worker（`gunicorn myapp:app --worker-class asgi`），无需 Uvicorn 也能跑 ASGI（https://gunicorn.org/asgi/）；因此「WSGI（gunicorn）/ ASGI（uvicorn）」的二分已过时。

**后果**：选 WSGI 框架（Flask）时，异步框架（FastAPI）那套 async 依赖注入、async ORM 的用法不适用；但运行时层面，gunicorn 两者都能跑。

### Python 大版本升级

老项目跨 Python 大版本前，先查官方 What's New 的 Removed 章节：https://docs.python.org/3/whatsnew/ 。重点看三类：stdlib 模块移除、`asyncio` 行为变更、`argparse` / `ast` 这类标准库接口变更。

**后果**：这些不是选型问题，是升级问题。选型阶段只需确认「目标框架支持的 Python 范围」，具体移除项在真正升级时按官方清单逐条核对。


## 生态惯例（非强制）

- **FastAPI + SQLAlchemy 是最常见的服务端组合**，因为它们同源（Tiangolo 生态共享 Pydantic）。惯例，非限制。**多源佐证**：SQLAlchemy 官方文档不点名 FastAPI（未找到非同一组织的官方背书）；FastAPI 官方教程以 SQLAlchemy 为例（https://fastapi.tiangolo.com/tutorial/sql-databases/），属于 Tiangolo 自家文档。
- **SQLModel 定位是「Pydantic + SQLAlchemy 的融合层」**，适合模型既要当请求校验体又要当数据库模型的场景。官方文档通篇使用同步 `Session` / `create_engine`，**未宣传 async API**——选它前需自行确认异步需求是否满足。<br>**版本绑定**：SQLModel 同时依赖 SQLAlchemy 与 Pydantic，选它会把这两个库的版本一起绑住——升级其中一个可能被 SQLModel 的约束挡住。选型时先查它的依赖声明（`pip show sqlmodel` 或 PyPI 的 `requires_dist`），再决定能不能接受这个绑定（https://sqlmodel.tiangolo.com/）。
- **Django 是「自带全家桶」路线**：ORM、迁移、认证、admin 都在框架内。与 FastAPI 的微框架路线是两种取向，选择取决于是否要开箱即用的 admin 与内容管理。
- **uv 的锁文件语义**：`uv.lock` 是 "a universal or cross-platform lockfile that captures the packages that would be installed across all possible Python markers such as operating system, architecture, and Python version"，且 "should be checked into version control"（https://docs.astral.sh/uv/concepts/projects/layout/）。Poetry 的 `poetry.lock` 类似："the locked dependencies in the poetry.lock file are the only ones present in the environment"（https://python-poetry.org/docs/managing-dependencies/）。两者都要入库。
- **Celery 不支持 Windows**："Celery is a project with minimal funding, so we don't support Microsoft Windows but it should be working."（https://pypi.org/project/celery/）。Windows 本地开发需要替代。
- **arq 已进入维护模式**，新项目不选它（https://pypi.org/project/arq/）。PyPI classifier 仍标 `Development Status :: 5 - Production/Stable`，但 description 写 "In maintenance only mode"（并引用 issue #510）；以 description 与仓库 issue 为准，不要看 classifier。Taskiq 仍标 Alpha，Dramatiq 与 Celery 活跃。
- **SQLModel / Tortoise / peewee 都是小而专的 ORM**，适合特定场景；没有 Alembic 级别的迁移生态兜底时，表结构演进要自己安排。

## 层之间怎么搭

```
Python 版本 → Web 框架 → 校验 → 数据库引擎 → ORM → 迁移工具 → 认证 → 依赖管理 → 测试 → 服务器模型(ASGI/WSGI) → 部署(打包 / 进程与反向代理) → 后台任务(可选)
```

两条主干路线：

**路线 A · 框架自带全家桶（Django）**
ORM、迁移、认证、admin 全部内置。一个进程一个项目，约定多、灵活少。适合内容型后台、管理系统、有复杂表单与权限的需求。

**路线 B · 微框架组合（FastAPI + SQLAlchemy + Alembic）**
每个层各选一个，组装自由，依赖显式。适合 API 优先、要精确控制异步模型或依赖组合的项目。

关键差异不在性能，而在**默认约定密度**：Django 给你一整套决定，FastAPI 给你一套选择。这条要写进决策表。

### 选框架会连带锁住的东西

- **Django 路线不需要单选 ORM、迁移、认证、依赖管理** —— 这四层由 Django 给出（内置 `makemigrations` / `migrate` 与认证系统）。决策表里这几层填「Django 内置」，不填具体库。
- **FastAPI 路线必须自己选 ORM 与迁移**，且两者强绑定：Alembic 官方定位就是 "for usage with the SQLAlchemy Database Toolkit"（https://alembic.sqlalchemy.org/）。选别的 ORM 就没有对应的迁移工具，需要另找或手写。
- **服务器模型由框架决定**：Flask 官方自述是 WSGI 框架，Litestar 与 Sanic 自述是 ASGI。FastAPI 是 ASGI（基于 Starlette）。选 WSGI 框架就用不了 async 依赖注入与 async ORM。
- **异步模型会传染到 ORM 层**：需要 async 的项目，Tortoise 的 async API 可用；SQLAlchemy 从 2.0 起提供 async 支持（官方 asyncio 扩展文档：https://docs.sqlalchemy.org/en/latest/orm/extensions/asyncio.html ，这是行为分界线，不是现值）；SQLModel 官方文档未宣传 async API（见硬约束节），选它前需自行确认。

### 数据库引擎的选法

见 `database-engines.md`（TS 轨与 Python 轨共用）：候选与官方定位、SQLite 并发写入限制、MongoDB 驱动选择（含 Motor 已弃用）、引擎与 ORM/认证/部署的耦合。

要点：**引擎选择与 ORM 选择是两个决定** —— SQLAlchemy 官方支持多种方言（https://docs.sqlalchemy.org/en/latest/dialects/），Tortoise 与 peewee 各有支持范围。引擎对部署复杂度的影响最大：PostgreSQL / MySQL / MongoDB 是独立服务，部署层要包含它们；SQLite 是进程内文件，只需备份文件。

### 认证层的选择差异

| 候选 | 定位 | 适合 |
|---|---|---|
| Django 内置认证 | 用户账号、用户组、权限、cookie 会话一体 | 走 Django 路线时直接用，不需另选 |
| Authlib | 构建 OAuth 与 OpenID Connect 的服务端与客户端 | 需要接第三方身份提供方（Google、GitHub、企业 SSO） |
| fastapi-users | 给 FastAPI 项目加注册与认证，可高度定制 | 走 FastAPI 路线且要自建用户体系 |

Django 官方对认证的完整表述："It handles user accounts, groups, permissions and cookie-based user sessions."（https://docs.djangoproject.com/en/stable/topics/auth/）——**权限与用户组是内置的**，这是 Django 路线的实质优势。
