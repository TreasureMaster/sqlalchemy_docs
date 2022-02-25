# Отображение классов Python

SQLAlchemy исторически имеет два разных стиля конфигурации mapper. Оригинальный API сопоставления обычно называют «классическим» стилем, тогда как более автоматизированный стиль сопоставления известен как «декларативный». SQLAlchemy теперь относится к этим двум стилям отображения как к императивному отображению (**imperative mapping**) и декларативному отображению (**declarative mapping**).

Оба стиля могут использоваться взаимозаменяемо, поскольку конечный результат каждого из них одинаков — определяемый пользователем класс, в котором [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) настроен для выбираемой единицы, обычно представленной объектом [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table).

Как императивное, так и декларативное сопоставление начинается с объекта реестра ORM [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry), который поддерживает набор сопоставленных классов. Этот реестр присутствует для всех отображений.

{% hint style="warning" %}
**Изменено в версии 1.4**: Декларативное и классическое сопоставление теперь называются «декларативным» и «императивным» сопоставлением и унифицированы внутри, все они происходят из конструкции реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry), представляющей набор связанных сопоставлений.
{% endhint %}

Полный набор стилей можно иерархически организовать следующим образом:

* [Declarative Mapping](otobrazhenie-klassov-python.md#undefined)
  * Использование базового класса [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base) с метаклассом
    * Declarative Table
    * Imperative Table (он же "гибридная таблица")
  * Использование декларативного декоратора [register.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped)
    * Declarative Table - комбинирование [registry.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped) с **\_\_tablename\_\_**
    * Императивная таблица (гибридная) - комбинирование [registry.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped) c **\_\_table\_\_**
    * Декларативное сопоставление с датаклассами и атрибутами attrs
      * Пример 1 - датаклассы с императивной таблицей
      * Пример 2 - датаклассы с декларативной таблицей
      * Пример 3 - attrs с императивной таблицей
* [Imperative (он же "классическое" сопоставление (mapping)](otobrazhenie-klassov-python.md#undefined)
  * Использование [registry.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively)
    * [Императивное сопоставление с датаклассами и attrs](otobrazhenie-klassov-python.md#undefined)

## Декларативное сопоставление (Declarative Mapping)

Декларативное сопоставление (**Declarative Mapping**) — это типичный способ создания сопоставлений в современной SQLAlchemy. Наиболее распространенный шаблон заключается в том, чтобы сначала создать базовый класс с помощью функции [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base), которая применит процесс декларативного отображения ко всем производным от него подклассам. Ниже представлена декларативная база, которая затем используется в декларативном сопоставлении таблиц:

```python
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import declarative_base

# декларативный базовый класс
Base = declarative_base()

# пример сопоставления, используя базовый класс
class User(Base):
    __tablename__ = 'user'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    fullname = Column(String)
    nickname = Column(String)
```

Выше вызываемый объект [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base) возвращает новый базовый класс, от которого могут наследоваться новые классы, подлежащие сопоставлению, поскольку выше создается новый сопоставленный класс **User**.

Базовый класс относится к объекту реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry), который поддерживает набор связанных сопоставленных классов. Функция [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base) на самом деле является сокращением для первого создания реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry) с помощью конструктора реестра, а затем создания базового класса с помощью метода [register.generate\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.generate\_base):

```python
from sqlalchemy.orm import registry

# эквивалентно Base = declarative_base()
# или Base = registry().generate_base()

mapper_registry = registry()
Base = mapper_registry.generate_base()
```

Реестр [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry) используется напрямую для доступа к различным стилям сопоставления в соответствии с различными вариантами использования. Основные стили отображения, предлагаемые реестром [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry), более подробно описаны в следующих разделах:

* Использование сгенерированного базового класса — декларативное сопоставление с использованием базового класса, сгенерированного объектом реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry).
* Декларативное сопоставление с использованием декоратора (без декларативной базы) — декларативное сопоставление с использованием декоратора, а не базового класса.
* [Императивное (также известное как классическое) сопоставление](otobrazhenie-klassov-python.md#imperativnoe-ono-zhe-klassicheskoe-sopostavlenie-imperative-mapping) - императивное сопоставление, определяющее все аргументы сопоставления напрямую, а не сканирование класса.

Документация по декларативному сопоставлению продолжается в разделе [Сопоставление классов с декларативным](otobrazhenie-klassov-v-deklarativnom-stile/).

{% hint style="info" %}
Смотри также:

[Сопоставление классов с Declarative](otobrazhenie-klassov-v-deklarativnom-stile/)

* Стили декларативного сопоставления
* Таблица конфигурации с Declarative
* Конфигурация сопоставления с Declarative
{% endhint %}

## Императивное (оно же классическое) сопоставление (Imperative mapping)

Императивное (**imperative**) или классическое (**classical**) сопоставление относится к конфигурации сопоставленного класса с использованием метода [register.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively), где целевой класс не включает никаких декларативных атрибутов класса. Стиль «императивной карты» исторически достигался с помощью функции [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper) напрямую, однако теперь эта функция ожидает наличия [sqlalchemy.orm.registry()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry).

{% hint style="danger" %}
**Устарело, начиная с версии 1.4**: прямое использование функции [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper) для достижения классического отображения устарело. Метод [registry.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively) сохраняет ту же функциональность, а также позволяет разрешать на основе строк другие сопоставленные классы из реестра.
{% endhint %}

В «классической» форме метаданные таблицы создаются отдельно с помощью конструкции [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), а затем связываются с классом **User** с помощью метода [register.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively):

```python
from sqlalchemy import Table, Column, Integer, String, ForeignKey
from sqlalchemy.orm import registry

mapper_registry = registry()

user_table = Table(
    'user',
    mapper_registry.metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(50)),
    Column('fullname', String(50)),
    Column('nickname', String(12))
)

class User:
    pass

mapper_registry.map_imperatively(User, user_table)
```

Информация о сопоставленных атрибутах, таких как отношения с другими классами, предоставляется через словарь свойств **properties**. В приведенном ниже примере показан второй объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), сопоставленный с классом **Address**, а затем связанный с пользователем **User** с помощью [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship):

```python
address = Table('address', metadata_obj,
            Column('id', Integer, primary_key=True),
            Column('user_id', Integer, ForeignKey('user.id')),
            Column('email_address', String(50))
            )

mapper_registry.map_imperatively(User, user, properties={
    'addresses' : relationship(Address, backref='user', order_by=address.c.id)
})

mapper_registry.map_imperatively(Address, address)
```

При использовании классических отображений классы должны предоставляться напрямую без использования системы «поиска строк», предоставляемой Declarative. Выражения SQL обычно указываются в терминах объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), т. е. `address.c.id` выше для отношения **Address**, а не `Address.id`, поскольку **Address** может еще не быть связан с метаданными таблицы, и мы не можем указать здесь строку.

В некоторых примерах в документации по-прежнему используется классический подход, но обратите внимание, что классический и декларативный подходы **полностью взаимозаменяемы**. Обе системы в конечном итоге создают одну и ту же конфигурацию, состоящую из таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), определяемого пользователем класса, связанного вместе с помощью [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper). Когда мы говорим о «поведении [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper)», это относится и к использованию декларативной системы — она все еще используется, просто за кулисами.

### Императивное сопоставление с датаклассами и attrs

Как описано в разделе «Декларативное сопоставление с классами данных и атрибутами», декоратор `@dataclass` и библиотека **attrs\_** работают как декораторы классов, которые сначала применяются к классу, прежде чем он будет передан SQLAlchemy для сопоставления. Точно так же, как мы можем использовать декоратор [registry.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped) для применения сопоставления в декларативном стиле к классу, мы также можем передать его методу [register.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively), чтобы мы могли императивно передать всю конфигурацию таблиц [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) и сопоставлений [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) в класс вместо того, чтобы определять их в самом классе как декларативные переменные класса:

```python
from __future__ import annotations

from dataclasses import dataclass
from dataclasses import field
from typing import List

from sqlalchemy import Column
from sqlalchemy import ForeignKey
from sqlalchemy import Integer
from sqlalchemy import MetaData
from sqlalchemy import String
from sqlalchemy import Table
from sqlalchemy.orm import registry
from sqlalchemy.orm import relationship

mapper_registry = registry()

@dataclass
class User:
    id: int = field(init=False)
    name: str = None
    fullname: str = None
    nickname: str = None
    addresses: List[Address] = field(default_factory=list)

@dataclass
class Address:
    id: int = field(init=False)
    user_id: int = field(init=False)
    email_address: str = None

metadata_obj = MetaData()

user = Table(
    'user',
    metadata_obj,
    Column('id', Integer, primary_key=True),
    Column('name', String(50)),
    Column('fullname', String(50)),
    Column('nickname', String(12)),
)

address = Table(
    'address',
    metadata_obj,
    Column('id', Integer, primary_key=True),
    Column('user_id', Integer, ForeignKey('user.id')),
    Column('email_address', String(50)),
)

mapper_registry.map_imperatively(User, user, properties={
    'addresses': relationship(Address, backref='user', order_by=address.c.id),
})

mapper_registry.map_imperatively(Address, address)
```

## Обзор конфигурации сопоставления (Mapper)

Со всеми формами сопоставления отображение класса можно настроить разными способами, передавая аргументы конструкции, которые становятся частью объекта [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper). Функция, которая в конечном итоге получает эти аргументы, — это функция [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper), которые доставляются ей из одной из внешних функций отображения, определенных в объекте реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry).

Существует четыре основных класса информации о конфигурации, которую ищет функция [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper):

### 1 Класс, который нужно сопоставить

Это класс, который мы создаем в нашем приложении. Как правило, нет никаких ограничений на структуру этого класса (1). Когда класс Python отображается, для класса может быть только **один** объект [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) (2).

При сопоставлении с [декларативным](otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping) стилем сопоставления класс, который должен быть сопоставлен, является либо подклассом декларативного базового класса, либо обрабатывается декоратором или функцией, такой как [register.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped).

При сопоставлении с [императивным](otobrazhenie-klassov-python.md#imperativnoe-ono-zhe-klassicheskoe-sopostavlenie-imperative-mapping) стилем класс передается непосредственно в качестве аргумента [map\_imperatively.class\_](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively.params.class\_).

{% hint style="success" %}
1. При работе под Python 2 класс «старого стиля» Python 2 является единственным несовместимым классом. При выполнении кода на Python 2 все классы должны наследоваться от класса **object** Python. В Python 3 это всегда так.
2. Существует устаревшая функция, известная как «не первичное сопоставление», когда дополнительные объекты сопоставления [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) могут быть связаны с классом, который уже сопоставлен, однако они не применяют инструментарий к классу. Эта функция устарела, начиная с SQLAlchemy 1.3.
{% endhint %}

### 2 Таблица или другой объект FromClause

В подавляющем большинстве случаев это экземпляр [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table). Для более продвинутых вариантов использования это может также относиться к любому типу объекта [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause), наиболее распространенными альтернативными объектами являются объект [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery) и [Join](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Join).

При отображении с помощью декларативного стиля сопоставления тематическая таблица либо создается декларативной системой на основе атрибута **\_\_tablename\_\_** и представленных объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), либо устанавливается с помощью атрибута **\_\_table\_\_**. Эти два стиля конфигурации представлены в **декларативной таблице** и **декларативной с императивной таблицей (также известной как гибридная декларативная)**.

При сопоставлении с [императивным](otobrazhenie-klassov-python.md#imperativnoe-ono-zhe-klassicheskoe-sopostavlenie-imperative-mapping) стилем предметная таблица передается позиционно в качестве аргумента [map\_imperatively.local\_table](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively.params.local\_table).

В отличие от требования «один преобразователь на класс» для отображаемого класса, таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) или другой объект [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause), который является предметом отображения, может быть связан с любым количеством отображений. [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) применяет изменения непосредственно к определяемому пользователем классу, но никоим образом не изменяет данную таблицу [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) или другое [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause).

### 3 Словарь свойств

Это словарь всех атрибутов, которые будут связаны с отображаемым классом. По умолчанию [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) создает записи для этого словаря, полученные из данной таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), в форме объектов [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty), каждый из которых ссылается на отдельный столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) отображаемой таблицы. Словарь свойств также будет содержать все другие типы объектов [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty), которые необходимо настроить, чаще всего экземпляры, сгенерированные конструкцией [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship).

При сопоставлении с помощью [декларативного](otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping) стиля сопоставления словарь свойств создается декларативной системой путем сканирования класса, который должен быть сопоставлен, на наличие соответствующих атрибутов. Заметки об этом процессе см. в разделе «**Определение сопоставленных свойств с декларативным**».

При сопоставлении с [императивным](otobrazhenie-klassov-python.md#imperativnoe-ono-zhe-klassicheskoe-sopostavlenie-imperative-mapping) стилем словарь свойств передается непосредственно в качестве аргумента свойств в [register.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively), который передает его вместе с параметром [mapper.properties](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.properties).

### 4 Другие параметры конфигурации сопоставления

Эти флаги задокументированы в [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper).

При сопоставлении с [декларативным](otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping) стилем сопоставления дополнительные аргументы конфигурации сопоставления настраиваются с помощью атрибута класса **\_\_mapper\_args\_\_**, задокументированного в **параметрах конфигурации сопоставления с декларативным стилем**.

При сопоставлении с [императивным](otobrazhenie-klassov-python.md#imperativnoe-ono-zhe-klassicheskoe-sopostavlenie-imperative-mapping) стилем аргументы ключевого слова передаются методу to [register.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively), который передает их функции [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper).

## Сопоставленное поведение класса

Для всех стилей сопоставления с использованием объекта реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry) характерно следующее поведение:

### 1 Конструктор по умолчанию

Реестр [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry) применяет конструктор по умолчанию, то есть метод **\_\_init\_\_**, ко всем отображаемым классам, которые явно не имеют собственного метода **\_\_init\_\_**. Поведение этого метода таково, что он предоставляет удобный конструктор ключевых слов, который будет принимать в качестве необязательных аргументов ключевых слов все именованные атрибуты. Например:

```python
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'user'

    id = Column(...)
    name = Column(...)
    fullname = Column(...)
```

Объект типа **User** выше будет иметь конструктор, который позволяет создавать объекты **User** как:

```python
u1 = User(name='some name', fullname='some fullname')
```

Вышеупомянутый конструктор можно настроить, передав вызываемый объект Python параметру [register.constructor](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.params.constructor), который обеспечивает желаемое поведение **\_\_init\_\_()** по умолчанию.

Конструктор также применяется к императивным отображениям:

```python
from sqlalchemy.orm import registry

mapper_registry = registry()

user_table = Table(
    'user',
    mapper_registry.metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(50))
)

class User:
    pass

mapper_registry.map_imperatively(User, user_table)
```

Вышеупомянутый класс, отображаемый императивно, как описано в [Imperative (также известном как Classical) Mappings](otobrazhenie-klassov-python.md#imperativnoe-ono-zhe-klassicheskoe-sopostavlenie-imperative-mapping), также будет иметь конструктор по умолчанию, связанный с реестром [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry).

{% hint style="info" %}
**Новое в версии 1.4**: классические сопоставления теперь поддерживают стандартный конструктор уровня конфигурации, когда они сопоставлены с помощью метода [register.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively).
{% endhint %}

### 2 Интроспекция классов Mapped и Mappers во время выполнения

Класс, сопоставленный с помощью реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry), также будет иметь несколько атрибутов, общих для всех сопоставлений:

**Атрибут \_\_mapper\_\_** будет ссылаться на [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper), связанный с классом:

```python
mapper = User.__mapper__
```

Этот [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) также является тем, что возвращается при использовании функции [inspect()](https://docs.sqlalchemy.org/en/14/core/inspection.html#sqlalchemy.inspect) для отображаемого класса:

```python
from sqlalchemy import inspect

mapper = inspect(User)
```

**Атрибут \_\_table\_\_** будет ссылаться на таблицу [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) или, в более общем смысле, на объект **FromClause**, на который сопоставляется классу:

```python
table = User.__table__
```

Этот **FromClause** также возвращается при использовании атрибута [Mapper.local\_table](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.local\_table) [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper):

```python
table = inspect(User).local_table
```

Для сопоставления наследования с одной таблицей, где класс является подклассом, не имеющим собственной таблицы, атрибут [Mapper.local\_table](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.local\_table), а также атрибут `.__table__` будет иметь значение `None`. Чтобы получить «selectable», который фактически выбран во время запроса для этого класса, это доступно через атрибут [Mapper.selectable](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.selectable):

```python
table = inspect(User).selectable
```

### 3 Особенности проверки Mapper

Как показано в предыдущем разделе, объект [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) доступен из любого сопоставленного класса, независимо от метода, с помощью системы [API Runtime Inspection](https://docs.sqlalchemy.org/en/14/core/inspection.html). Используя функцию [inspect()](https://docs.sqlalchemy.org/en/14/core/inspection.html#sqlalchemy.inspect), можно получить [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) из отображаемого класса:

```python
>>> from sqlalchemy import inspect
>>> insp = inspect(User)
```

Доступна подробная информация, включая [Mapper.columns](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.columns):

```python
>>> insp.columns
<sqlalchemy.util._collections.OrderedProperties object at 0x102f407f8>
```

Это пространство имен, которое можно просмотреть в формате списка или по отдельным именам:

```python
>>> list(insp.columns)
[Column('id', Integer(), table=<user>, primary_key=True, nullable=False), Column('name', String(length=50), table=<user>), Column('fullname', String(length=50), table=<user>), Column('nickname', String(length=50), table=<user>)]
>>> insp.columns.name
Column('name', String(length=50), table=<user>)
```

Другие пространства имен включают [Mapper.all\_orm\_descriptors](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.all\_orm\_descriptors), который включает в себя все сопоставленные атрибуты, а также гибриды, прокси-ассоциации:

```python
>>> insp.all_orm_descriptors
<sqlalchemy.util._collections.ImmutableProperties object at 0x1040e2c68>
>>> insp.all_orm_descriptors.keys()
['fullname', 'nickname', 'name', 'id']
```

А также [Mapper.column\_attrs](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.column\_attrs):

```python
>>> list(insp.column_attrs)
[<ColumnProperty at 0x10403fde0; id>, <ColumnProperty at 0x10403fce8; name>, <ColumnProperty at 0x1040e9050; fullname>, <ColumnProperty at 0x1040e9148; nickname>]
>>> insp.column_attrs.name
<ColumnProperty at 0x10403fce8; name>
>>> insp.column_attrs.name.expression
Column('name', String(length=50), table=<user>)
```

{% hint style="info" %}
**Смотри также:**

[Runtime Inspection API](https://docs.sqlalchemy.org/en/14/core/inspection.html)

[`Mapper`](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper)

[`InstanceState`](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.InstanceState)
{% endhint %}
