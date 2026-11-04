# Python 轨：层候选、硬约束、生态惯例

约束同样分两层：**技术硬约束**（官方文档可证实）与**生态惯例**（常见搭配，非强制）。

## 层候选

| 层 | 候选与官方定位 |
|---|---|
| Web 框架 | **FastAPI** — "a modern, fast (high-performance), web framework for building APIs with Python based on standard Python type hints."（https://fastapi.tiangolo.com/）<br>**Django** — "a high-level Python web framework that encourages rapid development and clean, pragmatic design."（https://www.djangoproject.com/）<br>**Flask** — "a lightweight WSGI web application framework."（https://flask.palletsprojects.com/en/stable/）<br>**Litestar** — "The Modern Python ASGI Framework Built to scale"；"Fast APIs with automatic OpenAPI documentation, dependency injection, type safety"（https://litestar.dev/）<br>**Sanic** — "a Python 3.10+ web server and web framework that's written to go fast."（https://sanic.dev/en/） |
| 数据层 | **SQLAlchemy 2.x** — 官方提供 async 支持（`sqlalchemy.ext.asyncio`，https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html）<br>**Django ORM** — Django 内置<br>**Tortoise ORM** — "an easy-to-use asyncio ORM inspired by Django"；"a lightweight, async-native Object-Relational Mapper"；"supports CPython 3.10 and later"（https://tortoise.github.io/）<br>**SQLModel** — "based on Python type annotations, and powered by Pydantic and SQLAlchemy"（https://sqlmodel.tiangolo.com/）<br>**peewee** — "a simple and small ORM"；"asyncio support built on the standard async drivers (aiosqlite, asyncpg, aiomysql)"（http://docs.peewee-orm.com/en/latest/） |
| 迁移 | **Alembic** — "a lightweight database migration tool for usage with the SQLAlchemy Database Toolkit for Python."（https://alembic.sqlalchemy.org/）<br>**Django makemigrations / migrate** — Django 内置 |
| 校验 | **Pydantic v2** — "Pydantic V2 is now the current production release of Pydantic"（https://docs.pydantic.dev/latest/migration） |
| 依赖管理 | **uv** — "An extremely fast Python package and project manager, written in Rust."；"A single tool to replace pip, pip-tools, pipx, poetry, pyenv, twine, virtualenv, and more."（https://docs.astral.sh/uv/）<br>**Poetry** — "Python packaging and dependency management made easy"（https://python-poetry.org/） |
| 测试 | **pytest** — "The pytest framework makes it easy to write small tests, yet scales to support complex functional testing"（https://pypi.org/project/pytest/）<br>**pytest-asyncio** — "a pytest plugin… facilitates testing of code that uses the asyncio library"（https://pypi.org/project/pytest-asyncio/）<br>**httpx** — "a fully featured HTTP client library for Python 3… provides both sync and async APIs"（https://pypi.org/project/httpx/） |
| 运行时与服务器 | **ASGI**（uvicorn / granian）、**WSGI**（gunicorn）—— 见硬约束节 |
| 后台任务 | **Celery** — "Distributed Task Queue."（https://pypi.org/project/celery/）<br>**Dramatiq** — "A fast and reliable distributed task processing library for Python 3."（https://pypi.org/project/dramatiq/）<br>**Taskiq** — "Distributed task queue with full async support"；分类器标注 `Development Status :: 3 - Alpha`（https://pypi.org/project/taskiq/）<br>**arq** — "Job queues in python with asyncio and redis"；官方标注 **"arq is in maintenance only mode"**（https://pypi.org/project/arq/） |
| 部署 | **Docker** — 官方 Python 镜像："The python images come in many flavors, each designed for a specific use case."；`python:<version>` 是默认镜像，另有 `-slim` / `-alpine`（https://github.com/docker-library/docs/blob/master/python/README.md） |

## 技术硬约束

### FastAPI 是 0.x，版本会破

官方原文："the current versions are still `0.x.x`, this reflects that each version could potentially have breaking changes. This follows the Semantic Versioning conventions."，并明确要求 "Pin your `fastapi` version"（https://fastapi.tiangolo.com/deployment/versions/）。

**后果**：FastAPI 不能用宽松范围，必须钉死具体版本或窄范围。

### FastAPI 与 Pydantic 的绑定

官方："Pydantic includes the tests for **FastAPI** with its own tests, so new versions of Pydantic (above `1.0.0`) are always compatible with FastAPI. You can pin `Pydantic` to any version above `1.0.0` that works for you. For example: `pydantic>=2.7.0,<3.0.0`"（同上）。

**后果**：FastAPI 项目里 Pydantic 版本是安全的，但仍需钉范围——因为 Pydantic v2 自身对 v1 有破坏性变更。

### Pydantic v1 → v2 的破坏性变更

官方迁移指南（https://docs.pydantic.dev/latest/migration）列出的重命名：`BaseModel` 方法改名为 `model_*`、`construct()` → `model_construct()`、`dict()` → `model_dump()`、`json()` → `model_dump_json()`、`parse_obj()` → `model_validate()`；`json_encoders` 不再推荐；`GetterDict` 与部分解析 API 行为变更。

保留旧 API 的方式：`from pydantic.v1 import ...`。

**后果**：跨 v1/v2 的项目升级时，这批 API 是主要破坏点。

### SQLAlchemy 1.4 → 2.0 的破坏性变更

官方迁移指南（https://docs.sqlalchemy.org/en/20/changelog/migration_20.html）列出的 "major APIs that are no longer present"：`Bound MetaData and connectionless execution`、`Emulated autocommit on Connection`、`The Session.autocommit parameter / mode`、`List / keyword arguments to select()`、`Python 2 support`。

另含：autocommit 从 Core 与 ORM 移除；ORM Query 与 Core Select 合并；`Query.get()` 移到 Session；关系加载改用属性而非字符串；`in_()` / `not_in()` 不再接受显式子查询；Result 行表现为 named tuple。

**后果**：1.4 项目升 2.0 必须先开 `RemovedIn20Warnings` 逐步清理（官方迁移步骤二、三步就是干这个）。

### Django 的 Python 版本矩阵

官方 FAQ 给出逐版本支持表（https://docs.djangoproject.com/en/6.1/faq/install/），例如 4.2 支持 3.8–3.12、5.0 支持 3.10–3.12、5.1 / 5.2 支持 3.10–3.13、6.0 / 6.1 支持 3.12–3.14。官方同时建议："Django recommends the latest Python 3 version"，且仅支持每个系列最新的微版本。

**后果**：Django 项目选 Python 版本要先查这张表，不能凭印象。

### Django 需要 WSGI 部署

官方 FAQ 记述 Django 生产部署走 WSGI（PEP 3333），开发用 `runserver`。

**后果**：Django + async 部署（ASGI）需要额外组件，不是内置路径。

### SQLite 的并发写入

官方 WAL 文档（https://www.sqlite.org/wal.html）：自 3.7.0 起支持 WAL，"Writers merely append new content to the end of the WAL file."；默认 rollback journal 模式下同一时刻通常只允许一个写入者。BEGIN CONCURRENT 等多写入者能力是扩展特性（https://sqlite.org/hctree/doc/begin-concurrent/doc/begin_concurrent.md）。

**后果**：SQLite 适合单机与开发，不适合多进程并发写入的生产场景。这是数据库选型的硬事实。

### MongoDB 的 Python 驱动与 ODM

官方建议直接用驱动："we recommend that new PyMongo users begin by working directly with the driver. PyMongo alone will meet most people's needs"（https://www.mongodb.com/docs/languages/python/pymongo-driver/current/integrations）。ODM 列表（MincePy、Ming、MongoEngine、MotorEngine、uMongo）标注为第三方、官方不维护。

**Motor 弃用原文**："As of May 14, 2025, Motor is deprecated in favor of the GA release of the PyMongo Async API in the PyMongo library… We strongly recommend migrating to the PyMongo Async API while Motor is still supported."（https://www.mongodb.com/docs/drivers/motor）

**后果**：新项目不要选 Motor。

### ASGI 与 WSGI 的分野

Flask 官方自述是 WSGI 框架；Litestar 与 Sanic 官方自述为 ASGI 框架。

**后果**：选 WSGI 框架（Flask）时，异步框架（FastAPI）那套 async 依赖注入、async ORM 的用法不适用。

### Python 3.13 / 3.14 的移除项

**3.13**（https://docs.python.org/3.13/whatsnew/3.13.html）：PEP 594 的 19 个 "dead batteries" 被移除——`aifc, audioop, cgi, cgitb, chunk, crypt, imghdr, mailcap, msilib, nis, nntplib, ossaudiodev, pipes, sndhdr, spwd, sunau, telnetlib, uu, xdrlib`；`2to3` 工具与 `lib2to3` 模块移除；`tkinter.tix` 移除。

**3.14**（https://docs.python.org/3.14/whatsnew/3.14.html）：`argparse` 的 `BooleanOptionalAction` 移除 `type/choices/metavar` 参数；`ast` 移除 `Bytes/Ellipsis/NameConstant/Num/Str` 类与 `Constant.n`/`Constant.s` 属性（改用 `Constant.value`）；`asyncio` 移除 `AbstractChildWatcher`/`FastChildWatcher`/`MultiLoopChildWatcher`/`PidfdChildWatcher`/`SafeChildWatcher`/`ThreadedChildWatcher` 及 `get_child_watcher`/`set_child_watcher` 系列；`asyncio.get_event_loop()` 在无当前循环时抛 `RuntimeError`。

**后果**：老项目升 Python 大版本时，这些是具体的报错点。

## 生态惯例（非强制）

- **FastAPI + SQLAlchemy + Alembic + Pydantic 是最常见的服务端组合**，因为它们同源（Tiangolo 生态共享 Pydantic）。惯例，非限制。
- **SQLModel 定位是「Pydantic + SQLAlchemy 的融合层」**，适合模型既要当请求校验体又要当数据库模型的场景。官方文档通篇使用同步 `Session` / `create_engine`，**未宣传 async API**——选它前需自行确认异步需求是否满足。
- **Django 是「自带全家桶」路线**：ORM、迁移、认证、admin 都在框架内。与 FastAPI 的微框架路线是两种取向，选择取决于是否要开箱即用的 admin 与内容管理。
- **uv 的锁文件语义**：`uv.lock` 是 "a universal or cross-platform lockfile that captures the packages that would be installed across all possible Python markers such as operating system, architecture, and Python version"，且 "should be checked into version control"（https://docs.astral.sh/uv/concepts/projects/layout/）。Poetry 的 `poetry.lock` 类似："the locked dependencies in the poetry.lock file are the only ones present in the environment"（https://python-poetry.org/docs/managing-dependencies/）。两者都要入库。
- **Celery 不支持 Windows**："Celery is a project with minimal funding, so we don't support Microsoft Windows but it should be working."（https://pypi.org/project/celery/）。Windows 本地开发需要替代。
- **arq 已进入维护模式**，新项目不选它（https://pypi.org/project/arq/）。Taskiq 仍标 Alpha，Dramatiq 与 Celery 活跃。
- **SQLModel / Tortoise / peewee 都是小而专的 ORM**，适合特定场景；没有 Alembic 级别的迁移生态兜底时，表结构演进要自己安排。

## 层之间怎么搭

```
Python 版本 → Web 框架 → 数据层(ORM) → 迁移工具 → 校验 → 依赖管理 → 测试 → 服务器模型(ASGI/WSGI) → 部署 → 后台任务(可选)
```

两条主干路线：

**路线 A · 框架自带全家桶（Django）**
ORM、迁移、认证、admin 全部内置。一个进程一个项目，约定多、灵活少。适合内容型后台、管理系统、有复杂表单与权限的需求。

**路线 B · 微框架组合（FastAPI + SQLAlchemy + Alembic）**
每个层各选一个，组装自由，依赖显式。适合 API 优先、要精确控制异步模型或依赖组合的项目。

关键差异不在性能，而在**默认约定密度**：Django 给你一整套决定，FastAPI 给你一套选择。这条要写进决策表。
