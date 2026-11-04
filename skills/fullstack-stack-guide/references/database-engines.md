# 数据库引擎选法（TS 轨与 Python 轨共用）

引擎选择与 ORM 选择是**两个决定**：ORM 通常支持多种引擎，但各有支持范围；选型时两者一起查 peer 范围。

## 候选与官方定位

| 引擎 | 官方定位原文 |
|---|---|
| PostgreSQL | "a powerful, open source object-relational database system with over 35 years of active development that has earned it a strong reputation for reliability, feature robustness, and performance."（https://www.postgresql.org/about/） |
| MySQL | "the world's most popular open source database. With its proven performance, reliability and ease-of-use, MySQL has become the leading database choice for web-based applications"（https://www.mysql.com/about/） |
| MariaDB | "a community-developed, commercially supported fork of the MySQL relational database management system (RDBMS), intended to remain free and open-source software under the GNU General Public License."（https://mariadb.org/about/） |
| SQLite | "a C-language library that implements a small, fast, self-contained, high-reliability, full-featured, SQL database engine. SQLite is the most used database engine in the world. SQLite is built into all mobile phones and most computers"（https://www.sqlite.org/） |
| MongoDB | "a document database designed for ease of application development and scaling."（https://www.mongodb.com/） |

## 怎么选

**SQLite 是嵌入式库，不是服务。** 官方定位是 "a C-language library"，直接读写普通磁盘文件，省掉了 client/server 架构。官方给出的适用范围："SQLite works great as the database engine for most low to medium traffic websites… any site that gets fewer than 100K hits/day should work fine with SQLite"（https://www.sqlite.org/whentouse.html）。

适合：内部工具、单机部署、开发环境、只读为主的分析场景。
不适合：多进程并发写入、跨机器共享数据、需要独立扩缩容。

**PostgreSQL 与 MySQL 是需要独立运维的服务。** 两者都是生产级关系库，选它们意味着多一个要部署、要监控、要做备份的组件。PostgreSQL 官方强调 "over 35 years of active development" 与特性健壮性；MySQL 官方强调易用与普及度。

**MongoDB 是文档数据库。** 结构不定的数据用它顺，但跨文档关系查询弱于关系库，事务支持也与关系库不同。选它意味着整个数据访问层按文档模型写，不是「换个连接串」。

## 硬约束

**SQLite 的并发写入。** 官方 WAL 文档说明 WAL 自 3.7.0 起可用，"Writers merely append new content to the end of the WAL file."；默认 rollback journal 模式下同一时刻通常只允许一个写入者。BEGIN CONCURRENT 等多写入者能力是扩展特性（https://sqlite.org/hctree/doc/begin-concurrent/doc/begin_concurrent.md）。

**结论**：SQLite 适合单机与开发，不适合多进程并发写入的生产场景。这是引擎本身的限制，不是 ORM 的。

**MongoDB 的 Python 驱动选择。** 官方建议直接用驱动："we recommend that new PyMongo users begin by working directly with the driver. PyMongo alone will meet most people's needs"（https://www.mongodb.com/docs/languages/python/pymongo-driver/current/integrations），ODM 列表（MincePy、Ming、MongoEngine、MotorEngine、uMongo）标注为第三方、官方不维护。

**Motor 已弃用**："As of May 14, 2025, Motor is deprecated in favor of the GA release of the PyMongo Async API… We strongly recommend migrating to the PyMongo Async API while Motor is still supported."（https://www.mongodb.com/docs/drivers/motor）

**TS 轨**：Prisma 与 Drizzle 都官方支持多种数据库，MongoDB 不构成 ORM 限制；Prisma 官方有 MongoDB connector（https://docs.prisma.io/docs/orm/v7/core-concepts/supported-databases/mongodb）与 SQLite connector（https://www.prisma.io/docs/orm/core-concepts/supported-databases/sqlite）。选型时以各 ORM 的官方支持列表为准，不以某个脚手架的矩阵为准。

**Python 轨**：SQLAlchemy 官方支持多种方言（https://docs.sqlalchemy.org/en/20/dialects/）；Tortoise 与 peewee 各有支持范围，查其官方文档。

## 与其它层的耦合

- **认证层**：Better Auth 官方说明 "Better Auth connects to a database to store data… The database will be used to store data such as users, sessions, and more."（https://www.better-auth.com/docs/concepts/database）——需要数据库支撑，也支持无数据库的 stateless session。选引擎时确认认证方案的存储要求。
- **迁移层**：Drizzle 的 dialect 与 Prisma 的 connector 都决定引擎可选项；Python 轨 Alembic 绑定 SQLAlchemy，SQLAlchemy 的方言支持决定引擎可选项。
- **部署层**：PostgreSQL / MySQL / MongoDB 是独立服务，部署层要包含它们；SQLite 是进程内文件，部署层只需备份文件。**这是选引擎对部署复杂度的最大影响。**
