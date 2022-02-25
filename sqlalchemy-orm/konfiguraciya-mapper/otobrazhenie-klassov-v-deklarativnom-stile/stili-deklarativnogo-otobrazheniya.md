# Стили декларативного отображения

Как было представлено в разделе «[Декларативное сопоставление](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping)», **Declarative Mapping** — это типичный способ создания сопоставлений в современной SQLAlchemy. В этом разделе будет представлен обзор форм, которые можно использовать для настройки декларативного преобразователя.

## Использование сгенерированного базового класса

Самый распространенный подход — создать «базовый» класс с помощью функции [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base):

```python
from sqlalchemy.orm import declarative_base

# декларативный базовый класс
Base = declarative_base()
```

Декларативный базовый класс также может быть создан из существующего реестра [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry) с помощью метода [register.generate\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.generate\_base):

```python
from sqlalchemy.orm import registry

reg = registry()

# декларативный базовый класс
Base = reg.generate_base()
```

В декларативном базовом классе новые сопоставленные классы объявляются как подклассы базового:

```python
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import declarative_base

# декларативный базовый класс
Base = declarative_base()

# пример сопоставления, использующего базовый класс
class User(Base):
    __tablename__ = 'user'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    fullname = Column(String)
    nickname = Column(String)
```

Выше функция [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base) возвращает новый базовый класс, от которого могут наследоваться новые классы, подлежащие сопоставлению, поскольку выше создается новый сопоставленный класс **User**.

Для каждого созданного подкласса тело класса затем следует подходу декларативного сопоставления, который определяет как таблицу [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), так и объект [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) за кулисами, которые составляют полное сопоставление.

{% hint style="info" %}
**Смотри также**:

[Table Configuration with Declarative](https://docs.sqlalchemy.org/en/14/orm/declarative\_tables.html)

[Mapper Configuration with Declarative](https://docs.sqlalchemy.org/en/14/orm/declarative\_config.html)
{% endhint %}

## Нединамическое создание явной базы (для использования с mypy, аналогично)

SQLAlchemy включает подключаемый [модуль Mypy](https://docs.sqlalchemy.org/en/14/orm/extensions/mypy.html), который автоматически приспосабливается к динамически сгенерированному базовому классу, предоставляемому функциями SQLAlchemy, такими как [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base). **Только для серии SQLAlchemy 1.4** этот подключаемый модуль работает вместе с новым набором заглушек ввода, опубликованных на [sqlalchemy2-stubs](https://pypi.org/project/sqlalchemy2-stubs).

Когда этот подключаемый модуль не используется или при использовании других инструментов [PEP 484](https://www.python.org/dev/peps/pep-0484), которые могут не знать, как интерпретировать этот класс, декларативный базовый класс может быть создан полностью явным образом с использованием непосредственно **DeclarativeMeta** следующим образом:

```python
from sqlalchemy.orm import registry
from sqlalchemy.orm.decl_api import DeclarativeMeta

mapper_registry = registry()

class Base(metaclass=DeclarativeMeta):
    __abstract__ = True

    registry = mapper_registry
    metadata = mapper_registry.metadata

    __init__ = mapper_registry.constructor
```

Вышеупомянутая база **Base** эквивалентна базе, созданной с помощью метода [register.generate\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.generate\_base), и будет полностью понята инструментами анализа типов без использования плагинов.

{% hint style="info" %}
Смотри также:

[Mypy/Pep-484 Support for ORM Mappings](https://docs.sqlalchemy.org/en/14/orm/extensions/mypy.html) — справочная информация о плагине Mypy, который автоматически применяет указанную выше структуру при запуске Mypy.
{% endhint %}

## Декларативное сопоставление с использованием декоратора (без декларативной базы)

В качестве альтернативы использованию «декларативного базового» класса можно явно применить декларативное отображение к классу, используя либо императивную технику, аналогичную «классическому» отображению, либо, более кратко, используя декоратор. Функция [register.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped) — это декоратор класса, который можно применить к любому классу Python без иерархии. В противном случае класс Python обычно настраивается в декларативном стиле:

```python
from sqlalchemy import Column, Integer, String, Text, ForeignKey

from sqlalchemy.orm import registry
from sqlalchemy.orm import relationship

mapper_registry = registry()

@mapper_registry.mapped
class User:
    __tablename__ = 'user'

    id = Column(Integer, primary_key=True)
    name = Column(String)

    addresses = relationship("Address", back_populates="user")

@mapper_registry.mapped
class Address:
    __tablename__ = 'address'

    id = Column(Integer, primary_key=True)
    user_id = Column(ForeignKey("user.id"))
    email_address = Column(String)

    user = relationship("User", back_populates="addresses")
```

Выше тот же реестр [registry](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry), который мы будем использовать для создания декларативного базового класса с помощью его метода [register.generate\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.generate\_base), также может применить сопоставление в декларативном стиле к классу без использования базы. При использовании вышеуказанного стиля сопоставление определенного класса будет продолжаться **только** в том случае, если декоратор применяется непосредственно к этому классу. Для отображения наследования декоратор следует применять к каждому подклассу:

```python
from sqlalchemy.orm import registry
mapper_registry = registry()

@mapper_registry.mapped
class Person:
    __tablename__ = "person"

    person_id = Column(Integer, primary_key=True)
    type = Column(String, nullable=False)

    __mapper_args__ = {

        "polymorphic_on": type,
        "polymorphic_identity": "person"
    }


@mapper_registry.mapped
class Employee(Person):
    __tablename__ = "employee"

    person_id = Column(ForeignKey("person.person_id"), primary_key=True)

    __mapper_args__ = {
        "polymorphic_identity": "employee"
    }
```

Оба стиля декларативного отображения «декларативная таблица» и «императивная таблица» могут использоваться с вышеуказанным стилем отображения.

Форма отображения декоратора особенно полезна при комбинировании декларативного отображения SQLAlchemy с другими формами объявления классов, особенно с модулем **dataclasses** классов данных Python. См. следующий раздел.

## Декларативное сопоставление с dataclasses и attrs

Модуль [dataclasses](https://docs.python.org/3/library/dataclasses.html), добавленный в Python 3.7, предоставляет декоратор класса `@dataclass` для автоматического создания стандартных определений методов **\_\_init\_\_()**, **\_\_eq\_\_()**, **\_\_repr\_\_()** и т. д. Еще одна очень популярная библиотека, которая делает то же самое и многое другое, — это [attrs](https://pypi.org/project/attrs/). Обе библиотеки используют декораторы классов для сканирования класса на наличие атрибутов, определяющих поведение класса, которые затем используются для создания методов, документации и аннотаций.

Декоратор класса [register.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped) позволяет декларативному отображению класса происходить после того, как класс был полностью сконструирован, что позволяет сначала обрабатывать класс другими декораторами класса. Таким образом, декораторы `@dataclass` и `@attr.s` могут быть применены первыми до того, как процесс отображения ORM продолжится с помощью декоратора [registry.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped) или метода [registry.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively), обсуждаемого в следующем разделе.

Сопоставление с `@dataclass` или `@attr.s` можно использовать простым способом с **декларативным стилем с императивной таблицей (также известным как гибридный декларативный)**, где таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), что означает, что она определяется отдельно и связана с классом через **\_\_table\_\_**. В частности, для классов данных также поддерживается **декларативная таблица**.

{% hint style="info" %}
**Новое в версии 1.4.0b2**: добавлена поддержка полного декларативного сопоставления при использовании классов данных.
{% endhint %}

Когда атрибуты определены с использованием **dataclasses**, декоратор `@dataclass` использует их, но оставляет на месте в классе. Процесс сопоставления SQLAlchemy, когда он встречает атрибут, который обычно должен быть сопоставлен со столбцом [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), явно проверяет, является ли атрибут частью установки классов данных, и если это так, **заменяет** атрибут класса данных, привязанный к классу, его обычными сопоставленными свойствами. Метод **\_\_init\_\_**, созданный `@dataclass`, остается нетронутым. Напротив, декоратор `@attr.s` фактически удаляет свои атрибуты, связанные с классом, после запуска декоратора, так что процесс сопоставления SQLAlchemy принимает эти атрибуты без каких-либо проблем.

{% hint style="info" %}
**Новое в версии 1.4**: добавлена поддержка прямого сопоставления классов данных Python, где [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) теперь будет обнаруживать атрибуты, относящиеся к модулю `@dataclasses`, и заменять их во время сопоставления, а не пропускать их, как это делается по умолчанию для любого атрибута класса, который не является частью отображения.
{% endhint %}

### Пример первый — классы данных с императивной таблицей

Пример сопоставления с использованием `@dataclass` с использованием **Declarative with Imperative Table (он же Hybrid Declarative)** выглядит следующим образом:

```python
from __future__ import annotations

from dataclasses import dataclass
from dataclasses import field
from typing import List
from typing import Optional

from sqlalchemy import Column
from sqlalchemy import ForeignKey
from sqlalchemy import Integer
from sqlalchemy import String
from sqlalchemy import Table
from sqlalchemy.orm import registry
from sqlalchemy.orm import relationship

mapper_registry = registry()


@mapper_registry.mapped
@dataclass
class User:
    __table__ = Table(
        "user",
        mapper_registry.metadata,
        Column("id", Integer, primary_key=True),
        Column("name", String(50)),
        Column("fullname", String(50)),
        Column("nickname", String(12)),
    )
    id: int = field(init=False)
    name: Optional[str] = None
    fullname: Optional[str] = None
    nickname: Optional[str] = None
    addresses: List[Address] = field(default_factory=list)

    __mapper_args__ = {   # type: ignore
        "properties" : {
            "addresses": relationship("Address")
        }
    }

@mapper_registry.mapped
@dataclass
class Address:
    __table__ = Table(
        "address",
        mapper_registry.metadata,
        Column("id", Integer, primary_key=True),
        Column("user_id", Integer, ForeignKey("user.id")),
        Column("email_address", String(50)),
    )
    id: int = field(init=False)
    user_id: int = field(init=False)
    email_address: Optional[str] = None
```

В приведенном выше примере атрибуты `User.id`, `Address.id` и `Address.user_id` определены как `field(init=False)`. Это означает, что параметры для них не будут добавлены в методы **\_\_init\_\_()**, но [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) по-прежнему сможет установить их после получения их значений во время сброса из автоинкремента или другого генератора значений по умолчанию. Чтобы разрешить их явное указание в конструкторе, вместо этого им будет присвоено значение по умолчанию `None`.

Чтобы [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) было объявлено отдельно, его необходимо указать непосредственно в словаре [Mapper.properties](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.params.properties), который сам указан в словаре **\_\_mapper\_args\_\_**, чтобы он был передан конструктору [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper). Альтернативой этому подходу является следующий пример.

### Пример второй — классы данных с декларативной таблицей

Полностью декларативный подход требует, чтобы объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) были объявлены как атрибуты класса, которые при использовании классов данных будут конфликтовать с атрибутами уровня класса данных. Подход к объединению их вместе заключается в использовании атрибута метаданных `metadata` в объекте `dataclass.field`, где может быть предоставлена информация о сопоставлении, специфичная для SQLAlchemy. Декларативное сопоставление поддерживает извлечение этих параметров, когда класс указывает атрибут `__sa_dataclass_metadata_key__`. Это также обеспечивает более краткий метод указания ассоциации [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship):

```python
from __future__ import annotations

from dataclasses import dataclass
from dataclasses import field
from typing import List

from sqlalchemy import Column
from sqlalchemy import ForeignKey
from sqlalchemy import Integer
from sqlalchemy import String
from sqlalchemy.orm import registry
from sqlalchemy.orm import relationship

mapper_registry = registry()


@mapper_registry.mapped
@dataclass
class User:
    __tablename__ = "user"

    __sa_dataclass_metadata_key__ = "sa"
    id: int = field(
        init=False, metadata={"sa": Column(Integer, primary_key=True)}
    )
    name: str = field(default=None, metadata={"sa": Column(String(50))})
    fullname: str = field(default=None, metadata={"sa": Column(String(50))})
    nickname: str = field(default=None, metadata={"sa": Column(String(12))})
    addresses: List[Address] = field(
        default_factory=list, metadata={"sa": relationship("Address")}
    )


@mapper_registry.mapped
@dataclass
class Address:
    __tablename__ = "address"
    __sa_dataclass_metadata_key__ = "sa"
    id: int = field(
        init=False, metadata={"sa": Column(Integer, primary_key=True)}
    )
    user_id: int = field(
        init=False, metadata={"sa": Column(ForeignKey("user.id"))}
    )
    email_address: str = field(
        default=None, metadata={"sa": Column(String(50))}
    )
```

#### Использование декларативных миксинов с классами данных

В разделе **Составление сопоставленных иерархий с миксинами** представлены декларативные классы миксинов. Одним из требований к декларативным миксинам является то, что определенные конструкции, которые не могут быть легко продублированы, должны быть заданы как вызываемые объекты с использованием декоратора [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr), например, в примере [Mixing in Relationships](https://docs.sqlalchemy.org/en/14/orm/declarative\_mixins.html#orm-declarative-mixins-relationships):

```python
class RefTargetMixin:
    @declared_attr
    def target_id(cls):
        return Column('target_id', ForeignKey('target.id'))

    @declared_attr
    def target(cls):
        return relationship("Target")
```

Эта форма поддерживается в объекте `field()` Dataclasses с помощью лямбда-выражения для указания конструкции SQLAlchemy внутри `field()`. Использование [declared\_attr()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr) для окружения лямбды необязательно. Если бы мы хотели создать наш класс **User** выше, где поля ORM были взяты из миксина, который сам является классом данных, форма была бы такой:

```python
@dataclass
class UserMixin:
    __tablename__ = "user"

    __sa_dataclass_metadata_key__ = "sa"

    id: int = field(
        init=False, metadata={"sa": Column(Integer, primary_key=True)}
    )

    addresses: List[Address] = field(
        default_factory=list, metadata={"sa": lambda: relationship("Address")}
    )

@dataclass
class AddressMixin:
    __tablename__ = "address"
    __sa_dataclass_metadata_key__ = "sa"
    id: int = field(
        init=False, metadata={"sa": Column(Integer, primary_key=True)}
    )
    user_id: int = field(
        init=False, metadata={"sa": lambda: Column(ForeignKey("user.id"))}
    )
    email_address: str = field(
        default=None, metadata={"sa": Column(String(50))}
    )

@mapper_registry.mapped
class User(UserMixin):
    pass

@mapper_registry.mapped
class Address(AddressMixin):
  pass
```

{% hint style="info" %}
**Новое в версии 1.4.2**: добавлена поддержка атрибутов миксина стиля «объявленный атрибут attr», а именно конструкций [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), а также объектов столбцов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) с объявлениями внешнего ключа, которые будут использоваться в сопоставлениях стиля «классы данных с декларативной таблицей».
{% endhint %}

### Пример третий — attrs с императивной таблицей

Отображение с использованием `@attr.s` в сочетании с императивной таблицей:

```python
import attr

# другие импорты

from sqlalchemy.orm import registry

mapper_registry = registry()


@mapper_registry.mapped
@attr.s
class User:
    __table__ = Table(
        "user",
        mapper_registry.metadata,
        Column("id", Integer, primary_key=True),
        Column("name", String(50)),
        Column("fullname", String(50)),
        Column("nickname", String(12)),
    )
    id = attr.ib()
    name = attr.ib()
    fullname = attr.ib()
    nickname = attr.ib()
    addresses = attr.ib()

# другие классы...
```

Сопоставления `@dataclass` и [attrs](https://pypi.org/project/attrs/) также могут использоваться с классическими сопоставлениями, т. е. с функцией [register.map\_imperatively()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.map\_imperatively). См. аналогичный пример в разделе «[Императивное сопоставление с классами данных и attrs](https://docs.sqlalchemy.org/en/14/orm/mapping\_styles.html#orm-imperative-dataclasses)».
