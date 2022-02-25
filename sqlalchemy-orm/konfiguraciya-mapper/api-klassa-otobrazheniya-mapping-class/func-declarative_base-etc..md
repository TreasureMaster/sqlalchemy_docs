# func declarative\_base(), etc.

## _function_ sqlalchemy.orm.declarative\_base(_bind=None_, _metadata=None_, _mapper=None_, _cls=\<class 'object'>_, _name='Base'_, _constructor=\<function \_declarative\_constructor>_, _class\_registry=None_, _metaclass=\<class 'sqlalchemy.orm.decl\_api.DeclarativeMeta'>_)

Создает базовый класс для декларативных определений классов.

Новому базовому классу будет предоставлен метакласс, который создает соответствующие объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) и выполняет соответствующие вызовы mapper() на основе информации, предоставленной декларативно в классе и любых подклассах класса.

Функция [declarative\_base()](func-declarative\_base-etc..md#function-sqlalchemy.orm.declarative\_base-bind-none-metadata-none-mapper-none-cls-less-than-class-obj) — это сокращенная версия использования метода [registry.generate\_base()](class-registry.md#method-sqlalchemy.orm.registry.generate\_base-mapper-none-cls-less-than-class-object-greater-than-nam). То есть следующее:

```python
from sqlalchemy.orm import declarative_base

Base = declarative_base()
```

Эквивалентно:

```python
from sqlalchemy.orm import registry

mapper_registry = registry()
Base = mapper_registry.generate_base()
```

См. строку документации для реестра [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl) и [registry.generate\_base()](class-registry.md#method-sqlalchemy.orm.registry.generate\_base-mapper-none-cls-less-than-class-object-greater-than-nam) для более подробной информации.

{% hint style="info" %}
**Изменено в версии 1.4**: Функция [declarative\_base()](func-declarative\_base-etc..md#function-sqlalchemy.orm.declarative\_base-bind-none-metadata-none-mapper-none-cls-less-than-class-obj) теперь является специализацией более общего класса [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl). Функция также перемещается в пакет `sqlalchemy.orm` из пакета `declarative.ext`.
{% endhint %}

#### Параметры:

* bind - Необязательный **Connectable** будет назначен атрибуту привязки экземпляра [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData).

{% hint style="warning" %}
**Устарело, начиная с версии 1.4**: аргумент **«bind»** с **declarative\_base** устарел и будет удален в **SQLAlchemy 2.0**.
{% endhint %}

* **metadata** - Необязательный экземпляр метаданных [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData). Все объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), неявно объявленные подклассами базы, будут совместно использовать эти метаданные. Экземпляр метаданных будет создан, если он не указан. Экземпляр [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) будет доступен через атрибут метаданных **metadata** сгенерированного декларативного базового класса.
* **mapper** - Необязательный вызываемый объект, по умолчанию используется mapper(). Будет использоваться для сопоставления подклассов с их таблицами.
* **cls** - По умолчанию **object**. Тип, используемый в качестве основы для сгенерированного декларативного базового класса. Может быть классом или кортежем классов.
* **name** - По умолчанию **Base**. Отображаемое имя созданного класса. Настройка этого не требуется, но может улучшить ясность трассировки и отладки.
* **constructor** - Укажите реализацию функции **\_\_init\_\_** в сопоставленном классе, который не имеет собственного **\_\_init\_\_**. По умолчанию используется реализация, которая назначает **\*\*kwargs** для объявленных полей и отношений экземпляру. Если указано `None`, **\_\_init\_\_** не будет предоставлено, и построение вернется к **cls.\_\_init\_\_** посредством обычной семантики Python.
* **class\_registry** - необязательный словарь, который будет служить **реестром имен классов-> сопоставленных классов**, когда строковые имена используются для идентификации классов внутри [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) и других. Позволяет двум или более декларативным базовым классам совместно использовать один и тот же реестр имен классов для упрощения межбазовых отношений.
* **metaclass** - По умолчанию используется **DeclarativeMeta**. Метакласс или совместимый с **\_\_metaclass\_\_** вызываемый для использования в качестве метатипа сгенерированного декларативного базового класса.

{% hint style="info" %}
Смотри также:

[registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl)
{% endhint %}

## _function_ sqlalchemy.orm.declarative\_mixin(_cls_)

Отмечает класс как предоставляющий функцию «декларативного миксина».

Например:

```python
from sqlalchemy.orm import declared_attr
from sqlalchemy.orm import declarative_mixin

@declarative_mixin
class MyMixin:

    @declared_attr
    def __tablename__(cls):
        return cls.__name__.lower()

    __table_args__ = {'mysql_engine': 'InnoDB'}
    __mapper_args__= {'always_refresh': True}

    id =  Column(Integer, primary_key=True)

class MyModel(MyMixin, Base):
    name = Column(String(1000))
```

В настоящее время декоратор [declarative\_mixin()](func-declarative\_base-etc..md#function-sqlalchemy.orm.declarative\_mixin-cls) никаким образом не модифицирует данный класс; его текущая цель состоит исключительно в том, чтобы помочь [плагину Mypy](https://docs.sqlalchemy.org/en/14/orm/extensions/mypy.html) идентифицировать декларативные классы миксинов SQLAlchemy, когда нет другого контекста.

{% hint style="warning" %}
**Новое в версии 1.4.6**.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Composing Mapped Hierarchies with Mixins](../otobrazhenie-klassov-v-deklarativnom-stile/sostavlenie-sopostavlennykh-ierarkhii-s-pomoshyu-miksinov.md)

[Using @declared\_attr and Declarative Mixins](https://docs.sqlalchemy.org/en/14/orm/extensions/mypy.html#mypy-declarative-mixins) - в документации [Mypy plugin documentation](https://docs.sqlalchemy.org/en/14/orm/extensions/mypy.html)
{% endhint %}

## _function_ sqlalchemy.orm.as\_declarative(_\*\*kw_)

Декоратор класса, который адаптирует данный класс к [declarative\_base()](func-declarative\_base-etc..md#function-sqlalchemy.orm.declarative\_base-bind-none-metadata-none-mapper-none-cls-less-than-class-obj).

Эта функция использует метод [registry.as\_declarative\_base()](class-registry.md#method-sqlalchemy.orm.registry.as\_declarative\_base-kw), сначала автоматически создавая реестр [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl), а затем вызывая декоратор.

Например:

```python
from sqlalchemy.orm import as_declarative

@as_declarative()
class Base(object):
    @declared_attr
    def __tablename__(cls):
        return cls.__name__.lower()
    id = Column(Integer, primary_key=True)

class MyMappedClass(Base):
    # ...
```

{% hint style="info" %}
Смотри также:

[registry.as\_declarative\_base()](class-registry.md#method-sqlalchemy.orm.registry.as\_declarative\_base-kw)
{% endhint %}

## _class_ sqlalchemy.orm.declared\_attr(_fget_, _cascading=False_)

Отмечает метод уровня класса как представляющий определение отображаемого свойства или специальное декларативное имя члена.

[declared\_attr](func-declarative\_base-etc..md#class-sqlalchemy.orm.declared\_attr-fget-cascading-false) обычно применяется в качестве декоратора к методу уровня класса, превращая атрибут в скалярное свойство, которое можно вызывать из неэкземплярного класса. Процесс декларативного сопоставления ищет эти вызываемые объекты **declared\_attr** при сканировании классов и предполагает, что любой атрибут, помеченный с помощью **declared\_attr**, будет вызываемым, который создаст объект, специфичный для декларативного сопоставления или конфигурации таблицы.

[declared\_attr](func-declarative\_base-etc..md#class-sqlalchemy.orm.declared\_attr-fget-cascading-false) обычно применим к миксинам, чтобы определить отношения, которые должны применяться к различным разработчикам класса. Он также используется для определения объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), включающих конструктор [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey), поскольку их нельзя легко повторно использовать в разных сопоставлениях. Пример ниже иллюстрирует оба:

```python
class ProvidesUser(object):
    "Миксин, который добавляет классам связь 'user'."

    @declared_attr
    def user_id(self):
        return Column(ForeignKey("user_account.id"))

    @declared_attr
    def user(self):
        return relationship("User")
```

[declared\_attr](func-declarative\_base-etc..md#class-sqlalchemy.orm.declared\_attr-fget-cascading-false) также может применяться к сопоставленным классам, например, для обеспечения «полиморфной» схемы наследования:

```python
class Employee(Base):
    id = Column(Integer, primary_key=True)
    type = Column(String(50), nullable=False)

    @declared_attr
    def __tablename__(cls):
        return cls.__name__.lower()

    @declared_attr
    def __mapper_args__(cls):
        if cls.__name__ == 'Employee':
            return {
                    "polymorphic_on":cls.type,
                    "polymorphic_identity":"Employee"
            }
        else:
            return {"polymorphic_identity":cls.__name__}
```

Чтобы использовать [declared\_attr](func-declarative\_base-etc..md#class-sqlalchemy.orm.declared\_attr-fget-cascading-false) внутри класса данных Python, как обсуждалось во [втором примере — классы данных с декларативной таблицей](../otobrazhenie-klassov-v-deklarativnom-stile/stili-deklarativnogo-otobrazheniya.md#primer-vtoroi-klassy-dannykh-s-deklarativnoi-tablicei), его можно поместить непосредственно в метаданные поля с помощью лямбда-выражения:

```python
@dataclass
class AddressMixin:
    __sa_dataclass_metadata_key__ = "sa"

    user_id: int = field(
        init=False, metadata={"sa": declared_attr(lambda: Column(ForeignKey("user.id")))}
    )
    user: User = field(
        init=False, metadata={"sa": declared_attr(lambda: relationship(User))}
    )
```

[declared\_attr](func-declarative\_base-etc..md#class-sqlalchemy.orm.declared\_attr-fget-cascading-false) также может быть опущен в этой форме, используя непосредственно лямбду, например:

```python
user: User = field(
    init=False, metadata={"sa": lambda: relationship(User)}
)
```

{% hint style="info" %}
Смотри также:

[Composing Mapped Hierarchies with Mixins](../otobrazhenie-klassov-v-deklarativnom-stile/sostavlenie-sopostavlennykh-ierarkhii-s-pomoshyu-miksinov.md) - иллюстрирует, как использовать декларативные миксины, которые являются основным вариантом использования для [declared\_attr](func-declarative\_base-etc..md#class-sqlalchemy.orm.declared\_attr-fget-cascading-false)

[Using Declarative Mixins with Dataclasses](../otobrazhenie-klassov-v-deklarativnom-stile/stili-deklarativnogo-otobrazheniya.md#ispolzovanie-deklarativnykh-miksinov-s-klassami-dannykh) - иллюстрирует специальные формы для использования с дата-классами Python
{% endhint %}

{% hint style="success" %}
Сигнатура класса

class sqlalchemy.orm.declared\_attr (`sqlalchemy.orm.base._MappedAttribute`, `builtins.property`)
{% endhint %}

## _function_ sqlalchemy.orm.has\_inherited\_table(_cls_)

Учитывая класс, верните `True`, если какой-либо из классов, от которых он наследуется, имеет сопоставленную таблицу, в противном случае верните `False`.

Это используется в декларативных миксинах для создания атрибутов, которые ведут себя по-разному для базового класса и подкласса в иерархии наследования.

{% hint style="info" %}
Смотри также:

[Controlling table inheritance with mixins](../otobrazhenie-klassov-v-deklarativnom-stile/sostavlenie-sopostavlennykh-ierarkhii-s-pomoshyu-miksinov.md#upravlenie-nasledovaniem-tablic-s-pomoshyu-primesei)
{% endhint %}

## _function_ sqlalchemy.orm.synonym\_for(_name_, _map\_column=False_)

Декоратор, создающий атрибут [synonym()](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.synonym) в сочетании с дескриптором Python.

Декорируемая функция передается в [synonym()](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.synonym) в качестве параметра [synonym.descriptor](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.synonym.params.descriptor):

```python
class MyClass(Base):
    __tablename__ = 'my_table'

    id = Column(Integer, primary_key=True)
    _job_status = Column("job_status", String(50))

    @synonym_for("job_status")
    @property
    def job_status(self):
        return "Status: %s" % self._job_status
```

Функция [hybrid properties](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#mapper-hybrids) SQLAlchemy обычно предпочтительнее, чем синонимы, которые являются более устаревшей функцией.

{% hint style="info" %}
Смотри также:

[Synonyms](../otobrazhenie-kolonok-i-vyrazhenii/izmenenie-povedeniya-atributov.md#sinonimy) - Обзор синонимов

[`synonym()`](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.synonym) - функция уровня mapper

[Using Descriptors and Hybrids](../otobrazhenie-kolonok-i-vyrazhenii/izmenenie-povedeniya-atributov.md#ispolzovanie-deskriptorov-i-gibridov) - Расширение **Hybrid Attribute** предлагает обновленный подход к расширению поведения атрибутов, более гибкий, чем это может быть достигнуто с помощью синонимов.
{% endhint %}
