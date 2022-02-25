# Табличная конфигурация в декларативном отображении

Как было представлено в разделе «[Декларативное сопоставление](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping)», декларативный стиль включает в себя возможность одновременного создания сопоставленного объекта [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) или непосредственного размещения таблицы или другого объекта [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause).

В следующих примерах предполагается декларативный базовый класс как:

```python
from sqlalchemy.orm import declarative_base

Base = declarative_base()
```

Все следующие примеры иллюстрируют класс, наследуемый от приведенного выше **Base**. Стиль декоратора, представленный в **декларативном сопоставлении с использованием декоратора (без декларативной основы)**, также полностью поддерживается всеми следующими примерами.

## Декларативная таблица

В декларативном базовом классе типичная форма отображения включает атрибут **\_\_tablename\_\_**, который указывает имя таблицы, которая должна быть сгенерирована вместе с отображением:

```python
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'user'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    fullname = Column(String)
    nickname = Column(String)
```

Выше объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) размещены в соответствии с определением класса. Процесс декларативного сопоставления сгенерирует новый объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) для коллекции [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), связанной с декларативной базой, и каждый указанный объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) станет частью коллекции [Table.columns](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table.columns) этого объекта [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table). Объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) могут опускать свое поле **«name»**, которое обычно является первым позиционным аргументом конструктора [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column); декларативная система назначит ключ, связанный с каждым столбцом, в качестве имени, чтобы создать таблицу, эквивалентную:

```python
# эквивалентный объект Table создан
user_table = Table(
    "user",
    Base.metadata,
    Column("id", Integer, primary_key=True),
    Column("name", String),
    Column("fullname", String),
    Column("nickname", String),
)
```

### Доступ к таблице и метаданным

Декларативно сопоставленный класс всегда будет включать атрибут с именем **\_\_table\_\_**; когда приведенная выше конфигурация с использованием **\_\_tablename\_\_** завершена, декларативный процесс делает таблицу [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) доступной через атрибут **\_\_table\_\_**:

```python
# доступ к таблице Table
user_table = User.__table__
```

Приведенная выше таблица, в конечном счете, та же самая, которая соответствует атрибуту [Mapper.local\_table](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.local\_table), который мы можем видеть через систему проверки во время выполнения ([runtime inspection system](https://docs.sqlalchemy.org/en/14/core/inspection.html)):

```python
from sqlalchemy import inspect

user_table = inspect(User).local_table
```

Коллекция метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), связанная как с декларативным реестром [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry), так и с базовым классом, часто необходима для выполнения операций **DDL**, таких как **CREATE**, а также для использования с инструментами миграции, такими как **Alembic**. Этот объект доступен через атрибут `.metadata` реестра, а также через декларативный базовый класс. Ниже для небольшого сценария мы можем захотеть создать **CREATE** для всех таблиц в базе данных SQLite:

```python
engine = create_engine("sqlite://")

Base.metadata.create_all(engine)
```

### Конфигурация декларативной таблицы

При использовании конфигурации декларативной таблицы с атрибутом декларативного класса **\_\_tablename\_\_** дополнительные аргументы, которые должны быть предоставлены конструктору таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), должны предоставляться с использованием атрибута декларативного класса **\_\_table\_args\_\_**.

Этот атрибут поддерживает как позиционные, так и ключевые аргументы, которые обычно отправляются конструктору таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table). Атрибут может быть указан в одной из двух форм. Один как словарь:

```python
class MyClass(Base):
    __tablename__ = 'sometable'
    __table_args__ = {'mysql_engine':'InnoDB'}
```

Другой, кортеж, где каждый аргумент является позиционным (обычно для ограничения constraints):

```python
class MyClass(Base):
    __tablename__ = 'sometable'
    __table_args__ = (
            ForeignKeyConstraint(['id'], ['remote_table.id']),
            UniqueConstraint('foo'),
            )
```

Аргументы ключевого слова можно указать в приведенной выше форме, указав последний аргумент как словарь:

```python
class MyClass(Base):
    __tablename__ = 'sometable'
    __table_args__ = (
            ForeignKeyConstraint(['id'], ['remote_table.id']),
            UniqueConstraint('foo'),
            {'autoload':True}
            )
```

Класс также может указать декларативный атрибут **\_\_table\_args\_\_**, а также атрибут **\_\_tablename\_\_** в динамическом стиле с помощью декоратора метода [declared\_attr()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr). См. раздел **Mixin и пользовательские базовые классы** для примеров того, как это часто используется.

### Явное имя схемы с декларативной таблицей

Имя схемы для таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), описанное в разделе **«Указание имени схемы»**, применяется к отдельной таблице с использованием аргумента [Table.schema](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table.params.schema). При использовании декларативных таблиц этот параметр передается, как и любой другой, в словарь **\_\_table\_args\_\_**:

```python
class MyClass(Base):
    __tablename__ = 'sometable'
    __table_args__ = {'schema': 'some_schema'}
```

Имя схемы также можно применить ко всем объектам таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) глобально с помощью параметра [MetaData.schema](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData.params.schema), описанного в разделе **«Указание имени схемы по умолчанию с метаданными»**. Объект [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) может быть создан отдельно и передан либо в [registry()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry), либо в [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base):

```python
from sqlalchemy import MetaData
metadata_obj = MetaData(schema="some_schema")

Base = declarative_base(metadata = metadata_obj)


class MyClass(Base):
    # будет использовать "some_schema" по умолчанию
    __tablename__ = 'sometable'
```

{% hint style="info" %}
Смотри также:

[Specifying the Schema Name](https://docs.sqlalchemy.org/en/14/core/metadata.html#schema-table-schema-name) - в документации [Describing Databases with MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html).
{% endhint %}

### Добавление новых колонок

Декларативная конфигурация таблицы позволяет добавлять новые объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) в двух сценариях. Самым простым является простое назначение новых объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) классу:

```python
MyClass.some_new_column = Column('data', Unicode)
```

Вышеупомянутая операция, выполненная над декларативным классом, который был сопоставлен с использованием декларативной базы (обратите внимание, а не декларативной формы декоратора), добавит указанный выше столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) в таблицу [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) с помощью метода [Table.append\_column()](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table.append\_column), а также добавит столбец в класс [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) для полного сопоставления.

{% hint style="success" %}
назначение новых столбцов существующему декларативно отображенному классу будет работать правильно только в том случае, если используется «декларативный базовый» класс, который также предоставляет управляемый метаклассом метод **\_\_setattr\_\_()**, который будет перехватывать эти операции. Это не сработает, если используется декларативный декоратор, предоставленный [register.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped), и не будет работать для императивно сопоставленного класса, сопоставленного с помощью [register.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively).
{% endhint %}

Другой сценарий, когда столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) добавляется на лету, — это когда наследующий подкласс, не имеющий собственной таблицы, указывает дополнительные столбцы; эти столбцы будут добавлены в таблицу суперкласса. В разделе **«Наследование одной таблицы»** обсуждается наследование одной таблицы.

## Декларативный с императивной таблицей (также известный как гибридный декларативный)

Декларативные сопоставления также могут быть предоставлены с уже существующим объектом [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) или, иначе, с [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) или другой произвольной конструкцией [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause) (такой как [Join](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Join) или [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery)), которая создается отдельно.

Это называется «гибридным декларативным» сопоставлением, поскольку класс сопоставляется с использованием декларативного стиля для всего, что связано с конфигурацией преобразователя, однако сопоставленный объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) создается отдельно и передается в декларативный процесс напрямую:

```python
from sqlalchemy.orm import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey


Base = declarative_base()

# построить таблицу Table напрямую.
# Коллекция Base.metadata обычно является
# хорошим выбором для метаданных,
# но можно использовать любую коллекцию метаданных.

user_table = Table(
    "user",
    Base.metadata,
    Column("id", Integer, primary_key=True),
    Column("name", String),
    Column("fullname", String),
    Column("nickname", String),
)

# построить класс User, используя эту таблицу.
class User(Base):
    __table__ = user_table
```

Выше объект таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) создается с использованием подхода, описанного в разделе **«Описание баз данных с метаданными**». Затем его можно применить непосредственно к классу, который декларативно сопоставлен. Атрибуты декларативного класса **\_\_tablename\_\_** и **\_\_table\_args\_\_** в этой форме не используются. Приведенная выше конфигурация часто более читабельна как встроенное определение:

```python
class User(Base):
    __table__ = Table(
        "user",
        Base.metadata,
        Column("id", Integer, primary_key=True),
        Column("name", String),
        Column("fullname", String),
        Column("nickname", String),
    )
```

Естественным эффектом вышеприведенного стиля является то, что атрибут **\_\_table\_\_** сам определяется в блоке определения класса. Таким образом, на него можно сразу ссылаться в последующих атрибутах, например, в приведенном ниже примере, который иллюстрирует обращение к столбцу типа в конфигурации полиморфного преобразователя:

```python
class Person(Base):
    __table__ = Table(
        'person',
        Base.metadata,
        Column('id', Integer, primary_key=True),
        Column('name', String(50)),
        Column('type', String(50))
    )

    __mapper_args__ = {
        "polymorphic_on": __table__.c.type,
        "polymorhpic_identity": "person"
    }
```

Форма «императивная таблица» также используется, когда необходимо сопоставить не-[Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) конструкцию, такую как объект [Join](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Join) или [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery). Пример ниже:

```python
from sqlalchemy import select, func

subq = select(
    func.count(orders.c.id).label('order_count'),
    func.max(orders.c.price).label('highest_order'),
    orders.c.customer_id
).group_by(orders.c.customer_id).subquery()

customer_select = select(customers, subq).join_from(
    customers, subq, customers.c.id == subq.c.customer_id
).subquery()

class Customer(Base):
    __table__ = customer_select
```

Справочную информацию о сопоставлении с не-[Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) конструкциями см. в разделах **Сопоставление класса с несколькими таблицами** и **Сопоставление класса с произвольными подзапросами**.

Форма «императивной таблицы» особенно полезна, когда сам класс использует альтернативную форму объявления атрибута, такую как классы данных Python. Подробнее см. в разделе [Декларативное сопоставление с классами данных и attrs](stili-deklarativnogo-otobrazheniya.md#deklarativnoe-sopostavlenie-s-dataclasses-i-attrs).

{% hint style="info" %}
Смотри также:

[Describing Databases with MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#metadata-describing)

[Declarative Mapping with Dataclasses and Attrs](https://docs.sqlalchemy.org/en/14/orm/declarative\_styles.html#orm-declarative-dataclasses)
{% endhint %}

## Декларативное сопоставление с отраженными таблицами

Доступно несколько шаблонов, которые обеспечивают создание сопоставленных классов для ряда объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), которые были проанализированы из базы данных, с использованием процесса отражения, описанного в разделе **«Отражение объектов базы данных»**.

Очень простой способ сопоставить класс с таблицей, отраженной из базы данных, — использовать декларативное гибридное сопоставление, передав параметр [Table.autoload\_with](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table.params.autoload\_with) в таблицу [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table):

```python
engine = create_engine("postgresql://user:pass@hostname/my_existing_database")

class MyClass(Base):
    __table__ = Table(
        'mytable',
        Base.metadata,
        autoload_with=engine
    )
```

Однако основным недостатком описанного выше подхода является то, что он требует присутствия источника подключения к базе данных во время объявления классов приложения; обычно классы объявляются как модули приложения, которые импортируются, но подключение к базе данных недоступно до тех пор, пока приложение не начнет выполнять код, чтобы оно могло использовать информацию о конфигурации и создавать механизм.

### Использование DeferredReflection

Для этого случая доступно простое расширение, называемое миксином [DeferredReflection](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.DeferredReflection), которое изменяет процесс декларативного сопоставления и откладывает его до тех пор, пока не будет вызван специальный метод уровня класса [DeferredReflection.prepare()](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.DeferredReflection.prepare), который будет выполнять процесс отражения для целевой базы данных. и интегрирует результаты с декларативным процессом сопоставления таблиц, то есть с классами, использующими атрибут **\_\_tablename\_\_**:

```python
from sqlalchemy.orm import declarative_base
from sqlalchemy.ext.declarative import DeferredReflection

Base = declarative_base()

class Reflected(DeferredReflection):
    __abstract__ = True

class Foo(Reflected, Base):
    __tablename__ = 'foo'
    bars = relationship("Bar")

class Bar(Reflected, Base):
    __tablename__ = 'bar'

    foo_id = Column(Integer, ForeignKey('foo.id'))
```

Выше мы создаем класс миксина **Reflected**, который будет служить базой для классов в нашей декларативной иерархии, которые должны отображаться при вызове метода `Reflected.prepare`. Приведенное выше сопоставление не будет завершено, пока мы не сделаем это, учитывая [Engine](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Engine):

```python
engine = create_engine("postgresql://user:pass@hostname/my_existing_database")
Reflected.prepare(engine)
```

Цель класса **Reflected** — определить область, в которой классы должны быть отображены отражающим образом. Плагин будет искать среди дерева подклассов цели, для которой вызывается `.prepare()`, и отображать все таблицы.

### Использование automap

Более автоматизированное решение для сопоставления с существующей базой данных, в которой должно использоваться отражение таблиц, заключается в использовании расширения [Automap](https://docs.sqlalchemy.org/en/14/orm/extensions/automap.html). Это расширение будет генерировать целые сопоставленные классы из схемы базы данных и позволяет несколько ловушек для настройки, включая возможность явного сопоставления некоторых или всех классов, при этом используя отражение для заполнения оставшихся столбцов.

{% hint style="info" %}
Смотри также:

[Automap](https://docs.sqlalchemy.org/en/14/orm/extensions/automap.html)
{% endhint %}
