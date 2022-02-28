# Описание

## Приступаем к работе

{% hint style="warning" %}
Пункты, записанные на английском языке с высокой долей вероятности имеют ссылки на оригинальную документацию SQLAlchemy, но могут быть исключения, если не удалось подобрать нормальный вариант перевода. То есть данные пункты отсутствуют в переводе.
{% endhint %}

Просмотр высокоуровневого описания и настройки.

* [Overview](https://docs.sqlalchemy.org/en/14/intro.html)
* [Installation Guide](https://docs.sqlalchemy.org/en/14/intro.html#installation)
* [Frequently Asked Questions](https://docs.sqlalchemy.org/en/14/faq/index.html)
* [Migration from 1.3](https://docs.sqlalchemy.org/en/14/changelog/migration\_14.html)
* [Glossary](https://docs.sqlalchemy.org/en/14/glossary.html)
* [Error Messages](https://docs.sqlalchemy.org/en/14/errors.html)
* [Changelog catalog](https://docs.sqlalchemy.org/en/14/changelog/index.html)

## Учебные пособия

### SQLAlchemy 1.4/2.0 переходный

SQLAlchemy 2.0 функционально доступен как часть SQLAlchemy 1.4 и более тесно интегрирует рабочие стили Core и ORM, чем когда-либо. В новом учебнике обе концепции представлены параллельно. Новые пользователи и те, кто начинает новые проекты, должны начать здесь!

* [Учебник по SQLAlchemy 1.4 / 2.0](sqlalchemy-2.0/uchebnik-po-sqlalchemy-1.4-2.0/) — основной учебник по SQLAlchemy 2.0
* [Migrating to SQLAlchemy 2.0](https://docs.sqlalchemy.org/en/14/changelog/migration\_20.html) — полная информация о переходе с 1.3 или 1.4 на 2.0

### Релизы SQLAlchemy 1.x

Учебник 1.x Object Relational и учебник Core — это **устаревшие** руководства, к которым следует обращаться для существующих кодовых баз SQLAlchemy.

* [Учебное пособие по реляционным объектам (API 1.x)](sqlalchemy-orm/uchebnoe-posobie-po-relyacionnym-obektam-1.x-api/)
* [SQL Expression Language Tutorial (1.x API)](https://docs.sqlalchemy.org/en/14/core/tutorial.html)

## Справочная документация

### SQLAlchemy ORM

* **Конфигурация ORM**: [Конфигурация Mapper](sqlalchemy-orm/konfiguraciya-mapper/) | [Конфигурация Relationship](sqlalchemy-orm/konfiguraciya-relationship/)
* **Использование ORM**: [Session Usage and Guidlines](https://docs.sqlalchemy.org/en/14/orm/session.html) | [Запрос данных, загрузка объектов](sqlalchemy-orm/zapros-query-dannykh-zagruzka-load-obektov/) |  [AsyncIO Support](https://docs.sqlalchemy.org/en/14/orm/extensions/asyncio.html)
* **Конфигурация расширений**: [Mypy integration](https://docs.sqlalchemy.org/en/14/orm/extensions/mypy.html) | [Association Proxy](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html) | [Hybrid Attributes](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html) | [Automap](https://docs.sqlalchemy.org/en/14/orm/extensions/automap.html) | [Mutable Scalars](https://docs.sqlalchemy.org/en/14/orm/extensions/mutable.html) | [All extensions](https://docs.sqlalchemy.org/en/14/orm/extensions/index.html)
* **Расширение ORM**: [ORM Events and Internals](https://docs.sqlalchemy.org/en/14/orm/extending.html)
* **Другое**: [Introduction to Examples](https://docs.sqlalchemy.org/en/14/orm/examples.html)

### SQLALchemy Core

* **Engines, Connections, Pools:** [Engine Configuration](https://docs.sqlalchemy.org/en/14/core/engines.html) | [Connections, Transactions, Results](https://docs.sqlalchemy.org/en/14/core/connections.html) | [AsyncIO Support](https://docs.sqlalchemy.org/en/14/orm/extensions/asyncio.html) | [Connection Pooling](https://docs.sqlalchemy.org/en/14/core/pooling.html)
* **Schema Definition:** [Overview](https://docs.sqlalchemy.org/en/14/core/schema.html) | [Tables and Columns](https://docs.sqlalchemy.org/en/14/core/metadata.html) | [Database Introspection (Reflection)](https://docs.sqlalchemy.org/en/14/core/reflection.html) | [Insert/Update Defaults](https://docs.sqlalchemy.org/en/14/core/defaults.html) | [Constraints and Indexes](https://docs.sqlalchemy.org/en/14/core/constraints.html) | [Using Data Definition Language (DDL)](https://docs.sqlalchemy.org/en/14/core/ddl.html)
* **SQL Reference:** [SQL Expression API docs](https://docs.sqlalchemy.org/en/14/core/expression\_api.html)
* **Datatypes:** [Overview](https://docs.sqlalchemy.org/en/14/core/types.html) | [Building Custom Types](https://docs.sqlalchemy.org/en/14/core/custom\_types.html#types-custom) | [API](https://docs.sqlalchemy.org/en/14/core/type\_api.html#types-api)
* **Core Basics:** [Overview](https://docs.sqlalchemy.org/en/14/core/api\_basics.html) | [Runtime Inspection API](https://docs.sqlalchemy.org/en/14/core/inspection.html) | [Event System](https://docs.sqlalchemy.org/en/14/core/event.html) | [Core Event Interfaces](https://docs.sqlalchemy.org/en/14/core/events.html) | [Creating Custom SQL Constructs](https://docs.sqlalchemy.org/en/14/core/compiler.html)

## Документация по диалектам

**Диалект** — это система, которую SQLAlchemy использует для связи с различными типами DBAPI и баз данных. В этом разделе описываются примечания, параметры и шаблоны использования для отдельных диалектов.

* [PostgreSQL](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html) | [MySQL](https://docs.sqlalchemy.org/en/14/dialects/mysql.html) | [SQLite](https://docs.sqlalchemy.org/en/14/dialects/sqlite.html) | [Oracle](https://docs.sqlalchemy.org/en/14/dialects/oracle.html) | [Microsoft SQL Server](https://docs.sqlalchemy.org/en/14/dialects/mssql.html)
* [More Dialects …](https://docs.sqlalchemy.org/en/14/dialects/index.html)
