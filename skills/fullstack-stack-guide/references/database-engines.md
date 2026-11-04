# 数据库引擎选法（TS 轨与 Python 轨共用）

引擎选择与 ORM 选择是**两个决定**：ORM 通常支持多种引擎，但各有支持范围；选型时两者一起查 peer 范围。

## 候选与官方定位

| 引擎 | 官方定位原文 |
|---|---|
| PostgreSQL | "a powerful, open source object-relational database system that uses and extends the SQL language combined with many features that safely store and scale the most complicated data workloads. The origins of PostgreSQL date back to 1986 as part of the POSTGRES project at the University of California at Berkeley and has nearly 40 years of active development on the core platform."（https://www.postgresql.org/about/） |
| MySQL | "the world's most popular open source database. With its proven performance, reliability and ease-of-use, MySQL has become the leading database choice for web-based applications"（https://www.mysql.com/about/） |
| MariaDB | "MariaDB Server is made by the original developers of MySQL and guaranteed to stay open source."（https://mariadb.org/about/） |
| SQLite | "a C-language library that implements a small, fast, self-contained, high-reliability, full-featured, SQL database engine. SQLite is the most used database engine in the world. SQLite is built into all mobile phones and most computers"（https://www.sqlite.org/） |
| MongoDB | "MongoDB is a document database designed to help developers build modern applications faster."（https://www.mongodb.com/docs/manual/） |

## 怎么选

**SQLite 是嵌入式库，不是服务。** 官方定位是 "a C-language library"，直接读写普通磁盘文件，省掉了 client/server 架构。官方给出的适用范围："SQLite works great as the database engine for most low to medium traffic websites… any site that gets fewer than 100K hits/day should work fine with SQLite"（https://www.sqlite.org/whentouse.html）。

适合：内部工具、单机部署、开发环境、只读为主的分析场景。
不适合：多进程并发写入、跨机器共享数据、需要独立扩缩容。

**PostgreSQL 与 MySQL 是需要独立运维的服务。** 两者都是生产级关系库，选它们意味着多一个要部署、要监控、要做备份的组件。PostgreSQL 官方强调 "nearly 40 years of active development on the core platform" 与特性健壮性；MySQL 官方强调易用与普及度。

**MongoDB 是文档数据库。** 结构不定的数据用它顺，但跨文档关系查询弱于关系库，事务支持也与关系库不同。选它意味着整个数据访问层按文档模型写，不是「换个连接串」。

## 硬约束

**SQLite 的并发写入。** 官方 WAL 文档说明 WAL 模式下写入者只需追加到 WAL 文件末尾；默认 rollback journal 模式下同一时刻通常只允许一个写入者。BEGIN CONCURRENT 等多写入者能力是扩展特性（https://sqlite.org/hctree/doc/begin-concurrent/doc/begin_concurrent.md）。WAL 模式的可用起始版本见官方文档。

**结论**：SQLite 适合单机与开发，不适合多进程并发写入的生产场景。这是引擎本身的限制，不是 ORM 的。

**MongoDB 的 Python 驱动选择。** 官方建议直接用驱动："we recommend that new PyMongo users begin by working directly with the driver. PyMongo alone will meet most people's needs"（https://www.mongodb.com/docs/languages/python/pymongo-driver/current/integrations），ODM 列表（MincePy、Ming、MongoEngine、MotorEngine、uMongo）标注为第三方、官方不维护。

**Motor 已弃用**："Motor is deprecated in favor of the GA release of the PyMongo Async API… We strongly recommend migrating to the PyMongo Async API while Motor is still supported."（https://www.mongodb.com/docs/drivers/motor）

**TS 轨**：Prisma 与 Drizzle 都官方支持多种数据库，MongoDB 不构成 ORM 限制；但 **Prisma 对 MongoDB 的支持有版本门槛**：官方 supported-databases 页会按当前大版本标注该连接器的就绪状态，未就绪时会指定改用哪个版本线。选型前必须先读该页顶部的状态说明：https://www.prisma.io/docs/orm/core-concepts/supported-databases/mongodb 。结论：不要因为「官网有 MongoDB 页面」就认为当前大版本可用。选型时以各 ORM 的官方支持列表为准，不以某个脚手架的矩阵为准。

**Python 轨**：SQLAlchemy 官方支持多种方言（https://docs.sqlalchemy.org/en/latest/dialects/）；Tortoise 与 peewee 各有支持范围，查其官方文档。

## 与其它层的耦合

- **认证层**：Better Auth 官方说明 "Better Auth connects to a database to store data… The database will be used to store data such as users, sessions, and more."（https://www.better-auth.com/docs/concepts/database）——需要数据库支撑，也支持无数据库的 stateless session。选引擎时确认认证方案的存储要求。
- **迁移层**：Drizzle 的 dialect 与 Prisma 的 connector 都决定引擎可选项；Python 轨 Alembic 绑定 SQLAlchemy，SQLAlchemy 的方言支持决定引擎可选项。
- **部署层**：PostgreSQL / MySQL / MongoDB 是独立服务，部署层要包含它们；SQLite 是进程内文件，部署层只需备份文件。**这是选引擎对部署复杂度的最大影响。**
