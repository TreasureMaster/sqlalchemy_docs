# Class registry

## _class_ sqlalchemy.orm.registry(_metadata=None_, _class\_registry=None_, _constructor=\<function \_declarative\_constructor>_, _\_bind=None_)

Обобщенный реестр для сопоставления классов.

Реестр **registry** служит основой для поддержания набора сопоставлений и предоставляет конфигурационные ловушки, используемые для сопоставления классов.

Поддерживаются три основных типа сопоставлений: **Declarative Base**, **Declarative Decorator** и **Imperative Mapping**. Все эти стили отображения могут использоваться взаимозаменяемо:

* [registry.generate\_base()](class-registry.md#method-sqlalchemy.orm.registry.generate\_base-mapper-none-cls-less-than-class-object-greater-than-nam) возвращает новый декларативный базовый класс и является базовой реализацией функции declarative\_base().
* registry.mapped() предоставляет декоратор класса, который будет применять декларативное сопоставление к классу без использования декларативного базового класса.
* registry.map\_imperatively() создаст Mapper для класса без сканирования класса на наличие атрибутов декларативного класса. Этот метод подходит для варианта использования, исторически предоставляемого классической функцией отображения mapper().

{% hint style="info" %}
**New in version 1.4.**
{% endhint %}

{% hint style="info" %}
Смотри также:

[Сопоставление классов Python](../otobrazhenie-klassov-python.md) — обзор стилей сопоставления классов.
{% endhint %}

### _method_ sqlalchemy.orm.registry.\_\_init\_\_(_metadata=None_, _class\_registry=None_, _constructor=\<function \_declarative\_constructor>_, _\_bind=None_)

Создать новый реестр [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl).

#### Параметры:

* **metadata** - Необязательный экземпляр метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData). Все объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), сгенерированные с помощью декларативного сопоставления таблиц, будут использовать эту коллекцию метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData). Если для этого аргумента оставить значение по умолчанию `None`, будет создана пустая коллекция метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData).
* **constructor** - Укажите реализацию функции **\_\_init\_\_** в сопоставленном классе, который не имеет собственного **\_\_init\_\_**. По умолчанию используется реализация, которая назначает **\*\*kwargs** для объявленных полей и отношений экземпляру. Если указано `None`, **\_\_init\_\_** не будет предоставлено, и построение вернется к **cls.\_\_init\_\_** посредством обычной семантики Python.
* **class\_registry** - необязательный словарь, который будет служить **реестром имен классов-> сопоставленных классов**, когда строковые имена используются для идентификации классов внутри [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) и других. Позволяет двум или более декларативным базовым классам совместно использовать один и тот же реестр имен классов для упрощения межбазовых отношений.

### _method_ sqlalchemy.orm.registry.as\_declarative\_base(_\*\*kw_)

Декоратор класса, который будет вызывать [register.generate\_base()](class-registry.md#method-sqlalchemy.orm.registry.generate\_base-mapper-none-cls-less-than-class-object-greater-than-nam) для заданного базового класса. Например:

```python
from sqlalchemy.orm import registry

mapper_registry = registry()

@mapper_registry.as_declarative_base()
class Base(object):
    @declared_attr
    def __tablename__(cls):
        return cls.__name__.lower()
    id = Column(Integer, primary_key=True)

class MyMappedClass(Base):
    # ...
```

Все аргументы ключевых слов, переданные в [registry.as\_declarative\_base()](class-registry.md#method-sqlalchemy.orm.registry.as\_declarative\_base-kw), передаются в [registry.generate\_base()](class-registry.md#method-sqlalchemy.orm.registry.generate\_base-mapper-none-cls-less-than-class-object-greater-than-nam).

### _method_ sqlalchemy.orm.registry.configure(_cascade=False_)

Настройте все еще не настроенные преобразователи mappers в этом реестре [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl).

Этап конфигурации используется для согласования и инициализации связей [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) между отображаемыми классами, а также для вызова событий конфигурации, таких как [MapperEvents.before\_configured()](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.MapperEvents.before\_configured) и [MapperEvents.after\_configured()](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.MapperEvents.after\_configured), которые могут использоваться расширениями ORM или определенными пользователем хуками расширений.&#x20;

Если один или несколько преобразователей в этом реестре содержат конструкции [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), которые ссылаются на сопоставленные классы в других реестрах, говорят, что этот реестр _**зависит**_ от этих реестров. Для автоматической настройки этих зависимых реестров флаг [configure.cascade](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.configure.params.cascade) должен быть установлен в значение `True`. В противном случае, если они не настроены, будет возбуждено исключение. Смысл такого поведения заключается в том, чтобы позволить приложению программно вызывать конфигурацию реестров, одновременно контролируя, достигает ли процесс неявно других реестров.

В качестве альтернативы вызову [register.configure()](class-registry.md#method-sqlalchemy.orm.registry.configure-cascade-false) можно использовать функцию ORM configure\_mappers(), чтобы убедиться, что конфигурация завершена для всех объектов реестра [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl) в памяти. Как правило, это проще в использовании, а также предшествует использованию объектов реестра в целом. Однако эта функция повлияет на все сопоставления во время выполнения процесса Python и может потребовать больше памяти/времени для приложения, которое использует множество реестров для различных целей, которые могут не понадобиться немедленно.

{% hint style="info" %}
Смотри также:

[`configure_mappers()`](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.configure\_mappers)
{% endhint %}

{% hint style="warning" %}
**Новое в версии 1.4.0b2.**
{% endhint %}

### _method_ sqlalchemy.orm.registry.dispose(_cascade=False_)

Удалите все преобразователи mappers в этом реестре [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl).

После вызова все классы, которые были сопоставлены в этом реестре, больше не будут иметь связанного с ними инструментария класса. Этот метод является аналогом общеприложенной функции clear\_mappers() для каждого реестра.

Если этот реестр содержит преобразователи, которые являются зависимостями от других реестров, как правило, через ссылки [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), то эти реестры также должны быть удалены. Когда такие реестры существуют по отношению к этому, также будет вызываться их метод [register.dispose()](class-registry.md#method-sqlalchemy.orm.registry.dispose-cascade-false), если для флага [dispose.cascade](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.dispose.params.cascade) установлено значение `True`; в противном случае возникает ошибка, если эти реестры еще не были удалены.

{% hint style="warning" %}
**Новое в версии 1.4.0b2.**
{% endhint %}

{% hint style="info" %}
Смотри также:

[`clear_mappers()`](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.clear\_mappers)
{% endhint %}

### _method_ sqlalchemy.orm.registry.generate\_base(_mapper=None_, _cls=\<class 'object'>_, _name='Base'_, _metaclass=\<class 'sqlalchemy.orm.decl\_api.DeclarativeMeta'>_)

Создайте декларативный базовый класс. Классы, которые наследуются от возвращаемого объекта класса, будут автоматически сопоставлены с использованием декларативного сопоставления. Например:

```python
from sqlalchemy.orm import registry

mapper_registry = registry()

Base = mapper_registry.generate_base()

class MyClass(Base):
    __tablename__ = "my_table"
    id = Column(Integer, primary_key=True)
```

Приведенный выше динамически сгенерированный класс эквивалентен приведенному ниже нединамическому примеру:

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

Метод [registry.generate\_base()](class-registry.md#method-sqlalchemy.orm.registry.generate\_base-mapper-none-cls-less-than-class-object-greater-than-nam) обеспечивает реализацию функции declarative\_base(), которая одновременно создает реестр [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl) и базовый класс.

См. раздел «[Декларативное сопоставление](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping)» для получения дополнительной информации и примеров.

#### Параметры:

* **mapper** - Необязательный вызываемый объект, по умолчанию используется mapper(). Эта функция используется для создания новых объектов Mapper.
* **cls** - По умолчанию **object**. Тип, используемый в качестве основы для сгенерированного декларативного базового класса. Может быть классом или кортежем классов.
* **name** - По умолчанию **Base**. Отображаемое имя созданного класса. Настройка этого не требуется, но может улучшить ясность трассировки и отладки.
* **metaclass** - По умолчанию используется **DeclarativeMeta**. Метакласс или совместимый с **\_\_metaclass\_\_** вызов, который можно использовать в качестве метатипа сгенерированного декларативного базового класса.

{% hint style="info" %}
Смотри также:

[Declarative Mapping](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping)

[`declarative_base()`](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base)
{% endhint %}

### _method_ sqlalchemy.orm.registry.map\_declaratively(_cls_)

Сопоставляет класс декларативно.

В этой форме сопоставления класс сканируется для получения информации о сопоставлении, включая столбцы, которые должны быть связаны с таблицей, и/или фактический объект таблицы.

Возвращает объект Mapper.

Например:

```python
from sqlalchemy.orm import registry

mapper_registry = registry()

class Foo:
    __tablename__ = 'some_table'

    id = Column(Integer, primary_key=True)
    name = Column(String)

mapper = mapper_registry.map_declaratively(Foo)
```

Эту функцию удобнее вызывать косвенно либо через декоратор класса registry.mapped(), либо путем создания подкласса декларативного метакласса, сгенерированного из [registry.generate\_base()](class-registry.md#method-sqlalchemy.orm.registry.generate\_base-mapper-none-cls-less-than-class-object-greater-than-nam).

Подробности и примеры см. в разделе «[Декларативное сопоставление](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping)».

#### Параметры:

* **cls** - класс для сопоставления.

#### Возвращает:

* объект Mapper

{% hint style="info" %}
Смотри также:

[Declarative Mapping](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping)

`registry.mapped()` - более общий интерфейс декоратора для этой функции.

``[`registry.map_imperatively()`](class-registry.md#method-sqlalchemy.orm.registry.map\_imperatively-class\_-local\_table-none-kw)``
{% endhint %}

### _method_ sqlalchemy.orm.registry.map\_imperatively(_class\__, _local\_table=None_, _\*\*kw_)

Императивно сопоставляет класс.

В этой форме сопоставления класс **не сканируется** на наличие какой-либо информации о сопоставлении. Вместо этого все конструкции сопоставления **передаются в качестве аргументов**.

Этот метод должен быть полностью эквивалентен классической функции mapper() SQLAlchemy, за исключением того, что он относится к конкретному реестру.

Например:

```python
from sqlalchemy.orm import registry

mapper_registry = registry()

my_table = Table(
    "my_table",
    mapper_registry.metadata,
    Column('id', Integer, primary_key=True)
)

class MyClass:
    pass

mapper_registry.map_imperatively(MyClass, my_table)
```

См. раздел «[Императивные (также известные как классические) сопоставления](../otobrazhenie-klassov-python.md#imperativnoe-ono-zhe-klassicheskoe-sopostavlenie-imperative-mapping)» для получения полной информации и примеров использования.

#### Параметры:

* **class\_** - Класс, который нужно сопоставить. Соответствует параметру mapper.class\_.
* **local\_table** - Таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) или другой объект [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause), являющийся предметом сопоставления. Соответствует параметру mapper.local\_table.
* **\*\*kw** - все остальные аргументы ключевого слова передаются непосредственно в функцию mapper().

{% hint style="info" %}
Смотри также:

[Imperative (a.k.a. Classical) Mappings](../otobrazhenie-klassov-python.md#imperativnoe-ono-zhe-klassicheskoe-sopostavlenie-imperative-mapping)

[Declarative Mapping](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping)
{% endhint %}

### _method_ sqlalchemy.orm.registry.mapped(_cls_)

Декоратор класса, который будет применять процесс декларативного сопоставления к данному классу.

Например:

```python
from sqlalchemy.orm import registry

mapper_registry = registry()

@mapper_registry.mapped
class Foo:
    __tablename__ = 'some_table'

    id = Column(Integer, primary_key=True)
    name = Column(String)
```

Подробности и примеры см. в разделе «[Декларативное сопоставление](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping)».

#### Параметры:

* **cls** - класс для сопоставления.

#### Возвращает:

* класс, который был пройден.

{% hint style="info" %}
Смотри также:

[Declarative Mapping](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping)

[registry.generate\_base()](class-registry.md#method-sqlalchemy.orm.registry.generate\_base-mapper-none-cls-less-than-class-object-greater-than-nam) - генерирует базовый класс, который автоматически применяет декларативное сопоставление к подклассам с помощью метакласса Python.
{% endhint %}

### _attribute_ sqlalchemy.orm.registry.mappers

Коллекция только для чтения всех объектов Mapper.
