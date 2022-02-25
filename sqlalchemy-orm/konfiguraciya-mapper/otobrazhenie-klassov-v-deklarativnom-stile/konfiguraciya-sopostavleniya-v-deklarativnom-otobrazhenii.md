# Конфигурация сопоставления в декларативном отображении

В разделе [Обзор конфигурации Mapper](../otobrazhenie-klassov-python.md#obzor-konfiguracii-sopostavleniya-mapper) обсуждаются общие элементы конфигурации конструкции [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper), которая представляет собой структуру, определяющую, как определенный пользовательский класс сопоставляется с таблицей базы данных или другой конструкцией SQL. В следующих разделах описываются конкретные детали того, как декларативная система создает [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper).

## Определение сопоставленных свойств с помощью декларативного

Примеры, приведенные в разделе [Конфигурация таблицы с декларативным](tablichnaya-konfiguraciya-v-deklarativnom-otobrazhenii.md), иллюстрируют сопоставление со столбцами, привязанными к таблице; сопоставление отдельного столбца с атрибутом класса ORM внутренне представлено конструкцией [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty). Существует множество других разновидностей свойств **mapper**, наиболее распространенной из которых является конструктор [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship). Другие виды свойств включают синонимы столбцов, которые определяются с помощью конструкции [synonym()](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.synonym), выражения SQL, которые определяются с помощью конструкции [column\_property()](https://docs.sqlalchemy.org/en/14/orm/mapping\_columns.html#sqlalchemy.orm.column\_property), а также отложенные столбцы и выражения SQL, которые загружаются только при доступе, определенные с помощью конструкции [deferred()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.deferred).

В то время как императивное сопоставление использует словарь свойств для установления всех атрибутов сопоставленного класса, при декларативном сопоставлении все эти свойства указываются в соответствии с определением класса, которые в случае декларативного сопоставления таблиц встроены в объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), которые будет использоваться для создания объекта [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table).

Работая с примером сопоставления пользователя **User** и адреса **Address**, мы можем проиллюстрировать декларативное сопоставление таблиц, которое включает не только объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), но также отношения и выражения SQL:

```python
# сопоставление атрибутов с использованием declarative с декларативной таблицей
# например __tablename__

from sqlalchemy import Column, Integer, String, Text, ForeignKey
from sqlalchemy.orm import column_property, relationship, deferred
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'user'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    firstname = Column(String(50))
    lastname = Column(String(50))

    fullname = column_property(firstname + " " + lastname)

    addresses = relationship("Address", back_populates="user")

class Address(Base):
    __tablename__ = 'address'

    id = Column(Integer, primary_key=True)
    user_id = Column(ForeignKey("user.id"))
    email_address = Column(String)
    address_statistics = deferred(Column(Text))

    user = relationship("User", back_populates="addresses")
```

Приведенное выше декларативное сопоставление таблиц содержит две таблицы, каждая из которых имеет [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), ссылающееся на другую, а также простое выражение SQL, отображаемое с помощью [column\_property()](https://docs.sqlalchemy.org/en/14/orm/mapping\_columns.html#sqlalchemy.orm.column\_property), и дополнительный столбец, который будет загружаться на «отложенной» основе, как определено конструкция [deferred()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.deferred). Дополнительную документацию по этим конкретным концепциям можно найти в **основных шаблонах отношений**, **использовании свойства column\_property** и **отложенной загрузке столбца**.

Свойства могут быть указаны с помощью декларативного отображения, как указано выше, с использованием стиля «гибридной таблицы»; объекты столбцов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), которые являются непосредственно частью таблицы, перемещаются в определение таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), но все остальное, включая составленные выражения SQL, по-прежнему будет встроено в определение класса. Конструкции, которые должны напрямую ссылаться на столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), будут ссылаться на него с точки зрения объекта таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table). Чтобы проиллюстрировать приведенное выше сопоставление с использованием стиля гибридной таблицы:

```python
# сопоставление атрибутов с использованием declarative с императивной таблицей
# например __table__

from sqlalchemy import Table
from sqlalchemy import Column, Integer, String, Text, ForeignKey
from sqlalchemy.orm import column_property, relationship, deferred
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class User(Base):
    __table__ = Table(
        "user",
        Base.metadata,
        Column("id", Integer, primary_key=True),
        Column("name", String),
        Column("firstname", String(50)),
        Column("lastname", String(50))
    )

    fullname = column_property(__table__.c.firstname + " " + __table__.c.lastname)

    addresses = relationship("Address", back_populates="user")

class Address(Base):
    __table__ = Table(
        "address",
        Base.metadata,
        Column("id", Integer, primary_key=True),
        Column("user_id", ForeignKey("user.id")),
        Column("email_address", String),
        Column("address_statistics", Text)
    )

    address_statistics = deferred(__table__.c.address_statistics)

    user = relationship("User", back_populates="addresses")
```

Что следует отметить выше:

* Таблица адресов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) содержит столбец с именем `address_statistics`, однако мы повторно сопоставляем этот столбец с тем же именем атрибута, чтобы он находился под контролем конструкции [deferred()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.deferred).
* Как при декларативном, так и при гибридном сопоставлении таблиц, когда мы определяем конструкцию [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey), мы всегда называем целевую таблицу, используя **имя таблицы**, а не имя сопоставленного класса.
* Когда мы определяем конструкции [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), поскольку эти конструкции создают связь между двумя отображаемыми классами, где один обязательно определяется перед другим, мы можем ссылаться на удаленный класс, используя его строковое имя. Эта функциональность также распространяется на область других аргументов, указанных в [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), таких как аргументы «основное соединение» и «упорядочить по». Подробнее об этом см. в разделе **«Поздняя оценка аргументов отношения»**.

## Параметры конфигурации Mapper с declarative

Со всеми формами сопоставления сопоставление класса настраивается с помощью параметров, которые становятся частью объекта [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper). Функция, которая в конечном итоге получает эти аргументы, — это функция [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper), и они доставляются ей из одной из внешних функций сопоставления, определенных в объекте реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry).

Для декларативной формы отображения аргументы преобразователя задаются с помощью декларативной переменной класса **\_\_mapper\_args\_\_**, которая представляет собой словарь, который передается в качестве аргументов ключевого слова в функцию [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper). Некоторые примеры:

### Столбец идентификатора ID версии

Параметры [mapper.version\_id\_col](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.version\_id\_col) и [mapper.version\_id\_generator](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.version\_id\_generator):

```python
from datetime import datetime

class Widget(Base):
    __tablename__ = 'widgets'

    id = Column(Integer, primary_key=True)
    timestamp = Column(DateTime, nullable=False)

    __mapper_args__ = {
        'version_id_col': timestamp,
        'version_id_generator': lambda v:datetime.now()
    }
```

### Наследование одной таблицы

Параметры [mapper.polymorphic\_on](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.polymorphic\_on) и [mapper.polymorphic\_identity](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.polymorphic\_identity):

```python
class Person(Base):
    __tablename__ = 'person'

    person_id = Column(Integer, primary_key=True)
    type = Column(String, nullable=False)

    __mapper_args__ = dict(
        polymorphic_on=type,
        polymorphic_identity="person"
    )

class Employee(Person):
    __mapper_args__ = dict(
        polymorphic_identity="employee"
    )
```

Словарь **\_\_mapper\_args\_\_** может быть сгенерирован из метода дескриптора, привязанного к классу, а не из фиксированного словаря, с использованием конструкции [declared\_attr()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr). В разделе **«Составление отображенных иерархий с помощью миксинов»** эта концепция обсуждается более подробно.

{% hint style="info" %}
Смотри также:

[Composing Mapped Hierarchies with Mixins](https://docs.sqlalchemy.org/en/14/orm/declarative\_mixins.html)
{% endhint %}

## Другие директивы декларативного отображения

### <mark style="color:orange;">\_\_declare\_last\_\_()</mark>

Хук **\_\_declare\_last\_\_()** позволяет определить функцию уровня класса, которая автоматически вызывается событием [MapperEvents.after\_configured()](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.MapperEvents.after\_configured), которое происходит после того, как предполагается, что сопоставления завершены и шаг «configure» завершен:

```python
class MyClass(Base):
    @classmethod
    def __declare_last__(cls):
        ""
        # сделать что-то с отображениями
```

### <mark style="color:orange;">\_\_declare\_first\_\_()</mark>

Аналогично **\_\_declare\_last\_\_()**, но вызывается в начале настройки **mapper** через событие [MapperEvents.before\_configured()](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.MapperEvents.before\_configured):

```python
class MyClass(Base):
    @classmethod
    def __declare_first__(cls):
        ""
        # сделать что-то, прежде чем сопоставления будут настроены
```

{% hint style="info" %}
**Новое в версии 0.9.3.**
{% endhint %}

### <mark style="color:orange;">metadata</mark>

Коллекция метаданных <mark style="color:orange;"></mark> [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), обычно используемая для назначения новой таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), представляет собой атрибут `registry.metadata`, связанный с используемым объектом реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry). При использовании декларативного базового класса, такого как созданный с помощью [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base), а также с помощью [register.generate\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.generate\_base), эти метаданные [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) также обычно присутствуют в виде атрибута с именем `.metadata`, который находится непосредственно в базовом классе и, следовательно, также в сопоставленном классе через наследование. Декларативный атрибут использует этот атрибут, если он присутствует, для определения целевой коллекции метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), а если он отсутствует, использует метаданные, связанные непосредственно с реестром [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry).

Этот атрибут также может быть назначен для того, чтобы повлиять на коллекцию метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), которая будет использоваться на основе сопоставленной иерархии для одной базы и/или реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry). Это вступает в силу независимо от того, используется ли декларативный базовый класс или если декоратор [registry.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped) используется напрямую, что позволяет использовать шаблоны, такие как базовый пример метаданных для абстрактного в следующем разделе, **\_\_abstract\_\_**. Аналогичный шаблон можно проиллюстрировать с помощью [register.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped) следующим образом:

```python
reg = registry()

class BaseOne:
    metadata = MetaData()

class BaseTwo:
    metadata = MetaData()

@reg.mapped
class ClassOne:
    __tablename__ = 't1'  # будет использовать reg.metadata

    id = Column(Integer, primary_key=True)

@reg.mapped
class ClassTwo(BaseOne):
    __tablename__ = 't1'  # будет использовать BaseOne.metadata

    id = Column(Integer, primary_key=True)

@reg.mapped
class ClassThree(BaseTwo):
    __tablename__ = 't1'  # будет использовать BaseTwo.metadata

    id = Column(Integer, primary_key=True)
```

{% hint style="warning" %}
**Изменено в версии 1.4.3**: декоратор [registry.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped) будет учитывать атрибут с именем **.metadata** в классе как альтернативную коллекцию метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), которая будет использоваться вместо метаданных, находящихся в самом реестре [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry). Это соответствует поведению базового класса, возвращаемого методом/функцией [register.generate\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.generate\_base) и [sqlalchemy.orm.declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base). Обратите внимание, что эта функция была нарушена из-за регрессии в 1.4.0, 1.4.1 и 1.4.2, даже при использовании [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base); 1.4.3 нужен для восстановления поведения.
{% endhint %}

{% hint style="info" %}
Смотри также:

[\_\_abstract\_\_](https://docs.sqlalchemy.org/en/14/orm/declarative\_config.html#declarative-abstract)
{% endhint %}

### \_\_abstract\_\_

**\_\_abstract\_\_** заставляет declarative полностью пропустить создание таблицы или преобразователя для класса. Класс можно добавить в иерархию так же, как миксин (см. **Миксин и пользовательские базовые классы**), позволяя подклассам расширяться только из специального класса:

```python
class SomeAbstractBase(Base):
    __abstract__ = True

    def some_helpful_method(self):
        ""

    @declared_attr
    def __mapper_args__(cls):
        return {"helpful mapper arguments":True}

class MyMappedClass(SomeAbstractBase):
    ""
```

Одним из возможных вариантов использования **\_\_abstract\_\_** является использование отдельных метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) для разных баз:

```python
Base = declarative_base()

class DefaultBase(Base):
    __abstract__ = True
    metadata = MetaData()

class OtherBase(Base):
    __abstract__ = True
    metadata = MetaData()
```

Выше классы, которые наследуются от **DefaultBase**, будут использовать одни метаданные [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) в качестве реестра таблиц, а те, которые наследуются от **OtherBase**, будут использовать другой. Затем сами таблицы могут быть созданы, возможно, в отдельных базах данных:

```python
DefaultBase.metadata.create_all(some_engine)
OtherBase.metadata.create_all(some_other_engine)
```

### \_\_table\_cls\_\_

Позволяет настраивать вызываемый объект/класс, используемый для создания таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table). Это очень открытый хук, который может позволить специальные настройки таблицы, которую вы генерируете здесь:

```python
class MyMixin(object):
    @classmethod
    def __table_cls__(cls, name, metadata_obj, *arg, **kw):
        return Table(
            "my_" + name,
            metadata_obj, *arg, **kw
        )
```

Приведенный выше миксин заставит все сгенерированные объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) включать префикс **«my\_**_**»**, за которым следует имя, обычно определяемое с использованием атрибута_ **\_\_tablename\_\_**.

**\_\_table\_cls\_\_** также поддерживает случай возврата `None`, что приводит к тому, что класс рассматривается как наследование одной таблицы по сравнению с его подклассом. Это может быть полезно в некоторых схемах настройки, чтобы определить, что наследование одной таблицы должно иметь место на основе аргументов для самой таблицы, например, определить как одиночное наследование, если первичный ключ отсутствует:

```python
class AutoTable(object):
    @declared_attr
    def __tablename__(cls):
        return cls.__name__

    @classmethod
    def __table_cls__(cls, *arg, **kw):
        for obj in arg[1:]:
            if (isinstance(obj, Column) and obj.primary_key) or \
                    isinstance(obj, PrimaryKeyConstraint):
                return Table(*arg, **kw)

        return None

class Person(AutoTable, Base):
    id = Column(Integer, primary_key=True)

class Employee(Person):
    employee_name = Column(String)
```

Приведенный выше класс **Employee** будет отображаться как однотабличное наследование по отношению к **Person**; столбец **employee\_name** будет добавлен в качестве члена таблицы **Person**.
