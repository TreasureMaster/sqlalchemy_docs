# Работа с метаданными базы данных

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Эта страница является частью [учебника по SQLAlchemy 1.4/2.0](./).

Предыдущий: [Работа с транзакциями и DBAPI](rabota-s-tranzakciyami-i-dbapi.md) | Далее: [Работа с данными](rabota-s-dannymi/)
{% endhint %}

Когда движки (engines) и выполнение SQL отключены, мы готовы начать некоторую алхимию. Центральным элементом как SQLAlchemy Core, так и ORM является язык выражений SQL, который позволяет свободно составлять компонуемые запросы SQL. Основой для этих запросов являются объекты Python, представляющие такие понятия базы данных, как таблицы и столбцы. Эти объекты вместе известны как [метаданные базы данных](https://docs.sqlalchemy.org/en/14/glossary.html#term-database-metadata).

Наиболее распространенные базовые объекты для метаданных базы данных в SQLAlchemy известны как метаданные [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) и столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column). В следующих разделах показано, как эти объекты используются как в стиле, ориентированном на Core, так и в стиле, ориентированном на ORM.

{% hint style="success" %}
Читатели ORM, оставайтесь с нами!

Как и в случае с другими разделами, пользователи Core могут пропустить разделы ORM, но пользователям ORM лучше всего ознакомиться с этими объектами с обеих точек зрения.
{% endhint %}

## Настройка метаданных с объектами таблицы

Когда мы работаем с реляционной базой данных, базовая структура, которую мы создаем и из которой делаем запросы, называется **таблицей**. В SQLAlchemy «таблица» представлена объектом Python с аналогичным названием [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table).

Чтобы начать использовать язык выражений SQLAlchemy, нам нужно создать объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), представляющие все таблицы базы данных, с которыми нам интересно работать. Каждая таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) может быть **объявлена**, то есть мы явно прописываем в исходном коде, как выглядит таблица, или может быть **отражена**, что означает, что мы генерируем объект на основе того, что уже присутствует в конкретной базе данных. Эти два подхода также можно комбинировать разными способами.

Независимо от того, будем ли мы объявлять или отражать наши таблицы, мы начинаем с коллекции, в которую мы будем помещать наши таблицы, известной как объект метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData). Этот объект, по сути, представляет собой [фасад](https://docs.sqlalchemy.org/en/14/glossary.html#term-facade) вокруг словаря Python, в котором хранится ряд объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), привязанных к их строковым именам. Построение этого объекта выглядит так:

```python
>>> from sqlalchemy import MetaData
>>> metadata_obj = MetaData()
```

Наличие одного объекта [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) для всего приложения является наиболее распространенным случаем, представленным в виде переменной уровня модуля в одном месте в приложении, часто в пакете типа «**models**» или «**dbschema**». Также может быть несколько коллекций метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), однако обычно наиболее полезно, если ряд объектов таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), связанных друг с другом, принадлежит одной коллекции метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData).

Когда у нас есть объект [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), мы можем объявить некоторые объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table). Это руководство начнется с классической учебной модели SQLAlchemy, модели пользователя таблицы **user**, которая, например, будет представлять пользователей веб-сайта, и таблицы адреса **address**, представляющего список адресов электронной почты, связанных со строками в таблице пользователей **user**. Обычно мы присваиваем каждому объекту [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) переменную, с помощью которой мы будем ссылаться на таблицу в коде приложения:

```python
>>> from sqlalchemy import Table, Column, Integer, String
>>> user_table = Table(
...     "user_account",
...     metadata_obj,
...     Column('id', Integer, primary_key=True),
...     Column('name', String(30)),
...     Column('fullname', String)
... )
```

Мы можем заметить, что приведенная выше конструкция таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) очень похожа на оператор SQL **CREATE TABLE**; начиная с имени таблицы, затем перечисляя каждый столбец, где каждый столбец имеет имя и тип данных. Объекты, которые мы используем выше:

* Таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) — представляет собой таблицу базы данных и присваивает себя коллекции метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData).
* Столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) — представляет столбец в таблице базы данных и присваивает себя объекту таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table). Столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) обычно включает имя строки и объект типа. Доступ к коллекции объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) с точки зрения родительской таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) обычно осуществляется через ассоциативный массив, расположенный в [Table.c](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table.c):

```python
>>> user_table.c.name
Column('name', String(length=30), table=<user_account>)

>>> user_table.c.keys()
['id', 'name', 'fullname']
```

* [Integer](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Integer), [String](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.String) — эти классы представляют типы данных SQL и могут быть переданы в столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) с созданием экземпляра или без него. Выше мы хотим указать длину «30» для столбца «**name**», поэтому мы создали экземпляр `String (30)`. Но для «**id**» и «**fullname**» мы их не указывали, поэтому можем отправить сам класс.

{% hint style="info" %}
Смотри также:

Справочная информация и документация по API для метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), таблиц [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) и столбцов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) находится в описании баз данных с метаданными ([Describing Databases with MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html)). Справочная документация по типам данных находится в столбцах и типах данных ([Column and Data Types](https://docs.sqlalchemy.org/en/14/core/types.html)).
{% endhint %}

## Объявление простых ограничений

Первый столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) в приведенной выше таблице **user\_table** включает параметр [Column.primary\_key](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.primary\_key), который является сокращенным методом указания того, что этот столбец должен быть частью первичного ключа для этой таблицы. Сам первичный ключ обычно объявляется неявно и представлен конструкцией [PrimaryKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.PrimaryKeyConstraint), которую мы можем видеть в атрибуте [Table.primary\_key](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table.primary\_key) объекта [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table):

```python
>>> user_table.primary_key
PrimaryKeyConstraint(
    Column(
        'id', Integer(),
        table=<user_account>,
        primary_key=True,
        nullable=False
    )
)
```

Ограничение, которое чаще всего объявляется явно, — это объект [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint), соответствующий ограничению [внешнего ключа базы данных](https://docs.sqlalchemy.org/en/14/glossary.html#term-foreign-key-constraint). Когда мы объявляем таблицы, которые связаны друг с другом, SQLAlchemy использует наличие этих объявлений ограничений внешнего ключа не только для того, чтобы они генерировались в операторах **CREATE** для базы данных, но и для помощи в построении выражений SQL.

Ограничение [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint), которое включает только один столбец в целевой таблице, обычно объявляется с использованием сокращенной записи на уровне столбца через объект [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey). Ниже мы объявляем вторую таблицу **address**, которая будет иметь ограничение внешнего ключа, ссылающееся на пользовательскую таблицу **user**:

```python
>>> from sqlalchemy import ForeignKey
>>> address_table = Table(
...     "address",
...     metadata_obj,
...     Column('id', Integer, primary_key=True),
...     Column('user_id', ForeignKey('user_account.id'), nullable=False),
...     Column('email_address', String, nullable=False)
... )
```

В приведенной выше таблице также представлен третий тип ограничения, которым в SQL является ограничение «**NOT NULL**», указанное выше с помощью параметра [Column.nullable](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.nullable).

{% hint style="info" %}
Совет:

При использовании объекта [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey) в определении столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) мы можем опустить тип данных для этого столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column); он автоматически выводится из связанного столбца, в приведенном выше примере тип данных [Integer](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Integer) столбца **user\_account.id**.
{% endhint %}

В следующем разделе мы создадим завершенный DDL для таблицы пользователей **user** и адресов **address**, чтобы увидеть завершенный результат.

## Отправка DDL в базу данных

Мы построили довольно сложную иерархию объектов для представления двух таблиц базы данных, начиная с корневого объекта [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), затем в два объекта [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), каждый из которых содержит набор объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) и [Constraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.Constraint). Эта объектная структура будет в центре большинства операций, которые мы будем выполнять как с Core, так и с ORM в будущем.

Первая полезная вещь, которую мы можем сделать с этой структурой, — это отправить операторы **CREATE TABLE** или [DDL](https://docs.sqlalchemy.org/en/14/glossary.html#term-DDL) в нашу базу данных SQLite, чтобы мы могли вставлять и запрашивать данные из них. У нас уже есть все инструменты, необходимые для этого, вызвав метод [MetaData.create\_all()](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData.create\_all) для наших метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), отправив ему [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine), который ссылается на целевую базу данных:

```python
>>> metadata_obj.create_all(engine)
```

```sql
BEGIN (implicit)
PRAGMA main.table_...info("user_account")
...
PRAGMA main.table_...info("address")
...
CREATE TABLE user_account (
    id INTEGER NOT NULL,
    name VARCHAR(30),
    fullname VARCHAR,
    PRIMARY KEY (id)
)
...
CREATE TABLE address (
    id INTEGER NOT NULL,
    user_id INTEGER NOT NULL,
    email_address VARCHAR NOT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY(user_id) REFERENCES user_account (id)
)
...
COMMIT
```

Процесс создания DDL по умолчанию включает некоторые специфичные для SQLite операторы **PRAGMA**, которые проверяют существование каждой таблицы перед отправкой **CREATE**. Полный набор шагов также включен в пару **BEGIN/COMMIT** для обеспечения транзакционного DDL (SQLite действительно поддерживает транзакционный DDL, однако драйвер базы данных **sqlite3** исторически запускает DDL в режиме автоматической фиксации **autocommit**).

Процесс создания также заботится об отправке операторов **CREATE** в правильном порядке; выше ограничение **FOREIGN KEY** зависит от существующей пользовательской таблицы **user**, поэтому таблица адресов **address** создается второй. В более сложных сценариях зависимостей ограничения **FOREIGN KEY** также могут применяться к таблицам постфактум с использованием **ALTER**.

Объект [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) также имеет метод [MetaData.drop\_all()](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData.drop\_all), который выдает операторы **DROP** в обратном порядке, как если бы он выдавал **CREATE** для удаления элементов схемы.

{% hint style="info" %}
**Инструменты миграции обычно подходят**

В целом, функция **CREATE/DROP** метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) полезна для наборов тестов, небольших и/или новых приложений, а также приложений, использующих недолговечные базы данных. Однако для управления схемой базы данных приложения в долгосрочной перспективе лучшим выбором, вероятно, будет инструмент управления схемой, такой как [Alembic](https://alembic.sqlalchemy.org), основанный на SQLAlchemy, поскольку он может управлять и координировать процесс постепенного изменения фиксированной схемы базы данных с течением времени, меняя дизайн приложения.
{% endhint %}

## Определение метаданных таблицы с помощью ORM

В этом разделе, посвященном только ORM, будет представлен пример объявления той же структуры базы данных, что и в предыдущем разделе, с использованием парадигмы конфигурации, более ориентированной на ORM. При использовании ORM процесс объявления метаданных таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) обычно сочетается с процессом объявления сопоставленных ([mapped](https://docs.sqlalchemy.org/en/14/glossary.html#term-mapped)) классов. Сопоставленный класс — это любой класс Python, который мы хотели бы создать, который затем будет иметь атрибуты, которые будут связаны со столбцами в таблице базы данных. Хотя существует несколько вариантов того, как это достигается, наиболее распространенный стиль известен как [декларативный](https://docs.sqlalchemy.org/en/14/orm/declarative\_config.html) и позволяет нам одновременно объявлять наши определяемые пользователем классы и метаданные таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table).

### Настройка реестра Registry

При использовании ORM коллекция метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) сохраняется, однако сама она содержится в объекте, предназначенном только для ORM, известном как реестр [registry](../../sqlalchemy-orm/konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl). Мы создаем реестр, построив его:

```python
>>> from sqlalchemy.orm import registry
>>> mapper_registry = registry()
```

Приведенный выше реестр [registry](../../sqlalchemy-orm/konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl) при создании автоматически включает объект [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), в котором будет храниться коллекция объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table):

```python
>>> mapper_registry.metadata
MetaData()
```

Вместо того, чтобы объявлять объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) напрямую, мы теперь будем объявлять их _**косвенно**_ через директивы, применяемые к нашим отображаемым классам. В наиболее распространенном подходе каждый отображаемый класс происходит от общего базового класса, известного как декларативная база (**declarative base**). Получаем новую декларативную базу из реестра [registry ](../../sqlalchemy-orm/konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl)с помощью метода [register.generate\_base()](../../sqlalchemy-orm/konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md#method-sqlalchemy.orm.registry.generate\_base-mapper-none-cls-less-than-class-object-greater-than-nam):

```python
>>> Base = mapper_registry.generate_base()
```

{% hint style="info" %}
**Совет**:

Шаги создания классов реестра [registry](../../sqlalchemy-orm/konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl) и «декларативной базы» можно объединить в один шаг с помощью исторически знакомой функции [declarative\_base()](../../sqlalchemy-orm/konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/func-declarative\_base-etc..md#function-sqlalchemy.orm.declarative\_base-bind-none-metadata-none-mapper-none-cls-less-than-class-obj):
{% endhint %}

```python
from sqlalchemy.orm import declarative_base
Base = declarative_base()
```

### Объявление сопоставленных (mapped) классов

Приведенный выше базовый объект **Base** — это класс Python, который будет служить базовым классом для классов отображения ORM, которые мы объявляем. Теперь мы можем определить сопоставленные классы ORM для таблицы пользователей **user** и адресов **address** в терминах новых классов **User** и **Address**:

```python
>>> from sqlalchemy.orm import relationship
>>> class User(Base):
...     __tablename__ = 'user_account'
...
...     id = Column(Integer, primary_key=True)
...     name = Column(String(30))
...     fullname = Column(String)
...
...     addresses = relationship("Address", back_populates="user")
...
...     def __repr__(self):
...        return f"User(id={self.id!r}, name={self.name!r}, fullname={self.fullname!r})"

>>> class Address(Base):
...     __tablename__ = 'address'
...
...     id = Column(Integer, primary_key=True)
...     email_address = Column(String, nullable=False)
...     user_id = Column(Integer, ForeignKey('user_account.id'))
...
...     user = relationship("User", back_populates="addresses")
...
...     def __repr__(self):
...         return f"Address(id={self.id!r}, email_address={self.email_address!r})"
```

Вышеупомянутые два класса теперь являются нашими сопоставленными классами и доступны для использования в персистентности ORM и операциях запросов, которые будут описаны позже. Но они также включают в себя объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), которые были сгенерированы как часть процесса декларативного сопоставления и эквивалентны тем, которые мы объявили непосредственно в предыдущем разделе Core. Мы можем увидеть эти объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) из декларативного сопоставленного класса, используя атрибут **.\_\_table\_\_**:

```python
>>> User.__table__
Table('user_account', MetaData(),
    Column('id', Integer(), table=<user_account>, primary_key=True, nullable=False),
    Column('name', String(length=30), table=<user_account>),
    Column('fullname', String(), table=<user_account>), schema=None)
```

Этот объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) был сгенерирован в результате декларативного процесса на основе атрибута **.\_\_tablename\_\_**, определенного для каждого из наших классов, а также посредством использования объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), назначенных атрибутам уровня класса внутри классов. Эти объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) обычно могут быть объявлены без явного поля «**name**» внутри конструктора, так как декларативный процесс назовет их автоматически на основе имени используемого атрибута.

{% hint style="info" %}
Смотри также:

[Декларативное сопоставление](../../sqlalchemy-orm/konfiguraciya-mapper/otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping) — обзор декларативного сопоставления классов
{% endhint %}

### Другие сведения о сопоставленном (mapped) классе

Для нескольких быстрых объяснений для классов выше, обратите внимание на следующие атрибуты:

* **классы имеют автоматически сгенерированный метод \_\_init\_\_()** - оба класса по умолчанию получают метод **\_\_init\_\_()**, который позволяет параметризованное построение объектов. Мы также можем предоставить собственный метод **\_\_init\_\_()**. **\_\_init\_\_()** позволяет нам создавать экземпляры имен атрибутов передачи **User** и **Address**, большинство из которых выше связаны непосредственно с объектами [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), как имена параметров:

```python
>>> sandy = User(name="sandy", fullname="Sandy Cheeks")
```

Подробнее об этом методе можно узнать в [конструкторе по умолчанию](../../sqlalchemy-orm/konfiguraciya-mapper/otobrazhenie-klassov-python.md#1-konstruktor-po-umolchaniyu).

* **мы предоставили метод \_\_repr\_\_()** — он _**полностью необязателен**_ и строго предназначен для того, чтобы наши пользовательские классы имели описательное строковое представление и в противном случае не требовались:

```python
>>> sandy
User(id=None, name='sandy', fullname='Sandy Cheeks')
```

Интересно отметить выше, что атрибут **id** автоматически возвращает `None` при доступе, а не вызывает **AttributeError**, как это было бы обычным поведением Python для отсутствующих атрибутов.

* **мы также включили двунаправленное отношение relationship** — это еще одна полностью необязательная конструкция, в которой мы использовали конструкцию ORM, называемую отношением [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) для обоих классов, которая указывает ORM, что эти классы **User** и **Address** ссылаются друг на друга в соотношении [один ко многим](https://docs.sqlalchemy.org/en/14/glossary.html#term-one-to-many) / [многие к одному](https://docs.sqlalchemy.org/en/14/glossary.html#term-many-to-one). Вышеупомянутое отношение [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) используется для того, чтобы мы могли продемонстрировать его поведение позже в этом руководстве; это **не требуется** для определения структуры таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table).

### Отправка DDL в базу данных

Этот раздел называется так же, как и раздел «[Передача DDL в базу данных](rabota-s-metadannymi-bazy-dannykh.md#otpravka-ddl-v-bazu-dannykh)», обсуждаемый с точки зрения Core. Это связано с тем, что отправка DDL с нашими сопоставленными классами ORM ничем не отличается. Если мы хотим создать DDL для объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), которые мы создали как часть наших декларативно сопоставленных классов, мы по-прежнему можем использовать [MetaData.create\_all()](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData.create\_all), как и раньше.

В нашем случае мы уже создали таблицы пользователей **user** и адресов **address** в нашей базе данных SQLite. Если бы мы этого еще не сделали, мы могли бы свободно использовать метаданные [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), связанные с нашим реестром [registry](../../sqlalchemy-orm/konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl), и декларативный базовый класс ORM, чтобы сделать это, используя [MetaData.create\_all()](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData.create\_all):

```python
# выдавать операторы CREATE с учетом реестра ORM
mapper_registry.metadata.create_all(engine)

# идентичный объект MetaData также присутствует в декларативной базе
Base.metadata.create_all(engine)
```

### Объединение объявлений базовой таблицы с декларативным ORM

В качестве альтернативного подхода к процессу сопоставления, показанному ранее в разделе «[Объявление сопоставленных классов](rabota-s-metadannymi-bazy-dannykh.md#obyavlenie-sopostavlennykh-mapped-klassov)», мы также можем использовать объекты таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), которые мы создали непосредственно в разделе «[Настройка метаданных с помощью объектов таблицы](rabota-s-metadannymi-bazy-dannykh.md#nastroika-metadannykh-s-obektami-tablicy)», в сочетании с декларативными сопоставленными классами из сгенерированного [declarative\_base()](../../sqlalchemy-orm/konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/func-declarative\_base-etc..md#function-sqlalchemy.orm.declarative\_base-bind-none-metadata-none-mapper-none-cls-less-than-class-obj) базового класса.

Эта форма называется [гибридной таблицей](../../sqlalchemy-orm/konfiguraciya-mapper/otobrazhenie-klassov-v-deklarativnom-stile/tablichnaya-konfiguraciya-v-deklarativnom-otobrazhenii.md#deklarativnyi-s-imperativnoi-tablicei-takzhe-izvestnyi-kak-gibridnyi-deklarativnyi), и она состоит в прямом присвоении атрибуту **.\_\_table\_\_**, а не в декларативном процессе:

```python
class User(Base):
    __table__ = user_table

     addresses = relationship("Address", back_populates="user")

     def __repr__(self):
        return f"User({self.name!r}, {self.fullname!r})"

class Address(Base):
    __table__ = address_table

     user = relationship("User", back_populates="addresses")

     def __repr__(self):
         return f"Address({self.email_address!r})"
```

Вышеупомянутые два класса эквивалентны тем, которые мы объявили в предыдущем примере сопоставления.

Традиционный «декларативный базовый» подход с использованием **\_\_tablename\_\_** для автоматического создания объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) остается наиболее популярным методом объявления метаданных таблицы. Однако, не принимая во внимание функциональность отображения ORM, которую он обеспечивает, поскольку объявление таблицы представляет собой просто синтаксическое удобство поверх конструктора таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table).

Затем мы обратимся к нашим сопоставленным классам ORM выше, когда будем говорить о манипулировании данными с точки зрения ORM, в разделе «[Вставка строк с помощью ORM](manipulyacii-s-dannymi-s-pomoshyu-orm.md)».

## Отражение таблицы

Чтобы завершить раздел о работе с метаданными таблицы, мы проиллюстрируем еще одну операцию, упомянутую в начале раздела, — отражение таблицы (**table reflection**). Отражение таблицы относится к процессу создания таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) и связанных объектов путем чтения текущего состояния базы данных. В то время как в предыдущих разделах мы объявляли объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) в Python, а затем передавали DDL в базу данных, процесс отражения делает это в обратном порядке.

В качестве примера отражения мы создадим новый объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), который представляет объект **some\_table**, созданный нами вручную в предыдущих разделах этого документа. Опять же, есть несколько вариантов того, как это выполняется, однако самым простым является создание объекта таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) с учетом имени таблицы и коллекции метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), к которой она будет принадлежать, а затем вместо указания отдельных объектов столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) и ограничения [Constraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.Constraint) передать его целевой [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine) с помощью параметра [Table.autoload\_with](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table.params.autoload\_with):

```python
>>> some_table = Table("some_table", metadata_obj, autoload_with=engine)
```

```sql
BEGIN (implicit)
PRAGMA main.table_...info("some_table")
[raw sql] ()
SELECT sql FROM  (SELECT * FROM sqlite_master UNION ALL   SELECT * FROM sqlite_temp_master) WHERE name = ? AND type = 'table'
[raw sql] ('some_table',)
PRAGMA main.foreign_key_list("some_table")
...
PRAGMA main.index_list("some_table")
...
ROLLBACK
```

В конце процесса объект **some\_table** теперь содержит информацию об объектах [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), присутствующих в таблице, и этот объект можно использовать точно так же, как и таблицу [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), которую мы объявили явно:

```python
>>> some_table
Table('some_table', MetaData(),
    Column('x', INTEGER(), table=<some_table>),
    Column('y', INTEGER(), table=<some_table>),
    schema=None)
```

{% hint style="info" %}
Смотри также:

Узнайте больше об отражении таблиц и схем в разделе Отражение объектов базы данных ([Reflecting Database Objects](https://docs.sqlalchemy.org/en/14/core/reflection.html)).

Для вариантов отражения таблиц, связанных с ORM, в разделе «[Декларативное сопоставление с отраженными таблицами](../../sqlalchemy-orm/konfiguraciya-mapper/otobrazhenie-klassov-v-deklarativnom-stile/tablichnaya-konfiguraciya-v-deklarativnom-otobrazhenii.md)» содержится обзор доступных параметров.
{% endhint %}

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел руководства: [Работа с данными](rabota-s-dannymi/)
{% endhint %}
