# Составление сопоставленных иерархий с помощью миксинов

Общая потребность при отображении классов с использованием [декларативного стиля](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping) заключается в совместном использовании некоторых функций, таких как набор общих столбцов, некоторые общие параметры таблицы или другие сопоставленные свойства, для многих классов. Стандартные идиомы Python для этого заключаются в том, чтобы классы наследуются от суперкласса, который включает эти общие функции.

При использовании декларативных отображений эта идиома допускается за счет использования классов примесей, а также за счет расширения декларативной базы, созданной либо методом [register.generate\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.generate\_base), либо функциями [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base).

Ниже приведен пример некоторых часто смешиваемых идиом:

```python
from sqlalchemy.orm import declarative_mixin
from sqlalchemy.orm import declared_attr

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

Там, где выше, класс **MyModel** будет содержать столбец **«id»** в качестве первичного ключа, атрибут **\_\_tablename\_\_**, который является производным от имени самого класса, а также **\_\_table\_args\_\_** и **\_\_mapper\_args\_\_**, определенные классом миксина **MyMixin**.

{% hint style="info" %}
Совет:

Использование декоратора класса [declarative\_mixin()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_mixin) помечает конкретный класс как предоставляющий услугу предоставления декларативных назначений SQLAlchemy в качестве примеси для других классов. Этот декоратор в настоящее время необходим только для того, чтобы подсказать подключаемому [модулю Mypy](https://docs.sqlalchemy.org/en/14/orm/extensions/mypy.html), что этот класс следует обрабатывать как часть декларативных сопоставлений.
{% endhint %}

Не существует фиксированного соглашения о том, предшествует ли **MyMixin** **Base** или нет. Применяются обычные правила разрешения методов Python, и приведенный выше пример будет работать так же хорошо с:

```python
class MyModel(Base, MyMixin):
    name = Column(String(1000))
```

Это работает, потому что здесь **Base** не определяет какие-либо переменные, которые определяет **MyMixin**, т. е. **\_\_tablename\_\_**, **\_\_table\_args\_\_**, **id** и т. д. Если бы **Base** действительно определял атрибут с тем же именем, класс, помещенный первым в списке наследования, определял бы, какой атрибут используется во вновь определенном классе.

## Дополнение базового класса Base

В дополнение к использованию чистого миксина, большинство методов в этом разделе также могут быть применены к самому базовому классу для шаблонов, которые должны применяться ко всем классам, производным от конкретной базы. Это достигается с помощью аргумента **cls** функции [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base):

```python
from sqlalchemy.orm import declared_attr

class Base:
    @declared_attr
    def __tablename__(cls):
        return cls.__name__.lower()

    __table_args__ = {'mysql_engine': 'InnoDB'}

    id =  Column(Integer, primary_key=True)

from sqlalchemy.orm import declarative_base

Base = declarative_base(cls=Base)

class MyModel(Base):
    name = Column(String(1000))
```

Где выше, **MyModel** и все другие классы, производные от **Base**, будут иметь имя таблицы, полученное из имени класса, столбец первичного ключа **id**, а также механизм **«InnoDB»** для MySQL.

## Миксины в колонках

Самый простой способ указать столбец в миксине — простое объявление:

```python
@declarative_mixin
class TimestampMixin:
    created_at = Column(DateTime, default=func.now())

class MyModel(TimestampMixin, Base):
    __tablename__ = 'test'

    id =  Column(Integer, primary_key=True)
    name = Column(String(1000))
```

Там, где выше, все декларативные классы, включающие **TimestampMixin**, также будут иметь столбец **created\_at**, который применяет метку времени ко всем вставкам строк.

Те, кто знаком с языком выражений SQLAlchemy, знают, что идентификатор объекта элементов предложения определяет их роль в схеме. Два объекта **Table** **a** и **b** могут иметь столбец с именем **id**, но они различаются тем, что `a.c.id` и `b.c.id` являются двумя разными объектами Python, ссылающимися на свои родительские таблицы **a** и **b** соответственно.

В случае со столбцом mixin создается впечатление, что явно создается только один объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), однако приведенный выше окончательный столбец **created\_at** должен существовать как отдельный объект Python для каждого отдельного целевого класса. Для этого декларативное расширение создает **копию** каждого объекта [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), встречающегося в классе, который определяется как миксин.

Этот механизм копирования ограничен простыми столбцами, не имеющими внешних ключей, так как сам [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey) содержит ссылки на столбцы, которые невозможно правильно воссоздать на этом уровне. Для столбцов с внешними ключами, а также для различных конструкций уровня преобразователя, требующих явного контекста назначения, предоставляется декоратор [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr), так что шаблоны, общие для многих классов, могут быть определены как вызываемые:

```python
from sqlalchemy.orm import declared_attr

@declarative_mixin
class ReferenceAddressMixin:
    @declared_attr
    def address_id(cls):
        return Column(Integer, ForeignKey('address.id'))

class User(ReferenceAddressMixin, Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
```

Где выше, вызываемый объект уровня класса **address\_id** выполняется в точке, в которой создается класс **User**, и декларативное расширение может использовать результирующий объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), возвращенный методом, без необходимости его копирования.

На столбцы, созданные с помощью [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr), также может ссылаться **\_\_mapper\_args\_\_** в ограниченной степени, в настоящее время `polymorphic_on` и `version_id_col`; декларативное расширение разрешит их во время создания класса:

```python
@declarative_mixin
class MyMixin:
    @declared_attr
    def type_(cls):
        return Column(String(50))

    __mapper_args__= {'polymorphic_on':type_}

class MyModel(MyMixin, Base):
    __tablename__='test'
    id =  Column(Integer, primary_key=True)
```

## Миксины в отношениях relationship

Отношения, созданные с помощью [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), предоставляются с декларативными классами примесей исключительно с использованием подхода [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr), что устраняет любую двусмысленность, которая может возникнуть при копировании отношения и его содержимого, возможно связанного со столбцами. Ниже приведен пример, который сочетает в себе столбец внешнего ключа и связь, так что два класса **Foo** и **Bar** могут быть настроены для ссылки на общий целевой класс через «многие к одному»:

```python
@declarative_mixin
class RefTargetMixin:
    @declared_attr
    def target_id(cls):
        return Column('target_id', ForeignKey('target.id'))

    @declared_attr
    def target(cls):
        return relationship("Target")

class Foo(RefTargetMixin, Base):
    __tablename__ = 'foo'
    id = Column(Integer, primary_key=True)

class Bar(RefTargetMixin, Base):
    __tablename__ = 'bar'
    id = Column(Integer, primary_key=True)

class Target(Base):
    __tablename__ = 'target'
    id = Column(Integer, primary_key=True)
```

### Использование расширенных аргументов отношений (например, primaryjoin и т. д.)

Определения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), которые требуют явных выражений _**primaryjoin**_, _**order\_by**_ и т. д., должны во всех случаях, кроме самых упрощенных, использовать формы поздней привязки (**late bound**) для этих аргументов, то есть использовать либо строковую форму, либо функцию/лямбда. Причина этого в том, что связанные объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), которые должны быть настроены с использованием `@declared_attr`, недоступны для другого атрибута `@declared_attr`; в то время как методы будут работать и возвращать новые объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), это не объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), которые Declarative будет использовать, поскольку он вызывает методы самостоятельно, таким образом используя _разные_ объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column).

Каноническим примером является условие _**primaryjoin**_, которое зависит от другого смешанного столбца:

```python
@declarative_mixin
class RefTargetMixin:
      @declared_attr
      def target_id(cls):
          return Column('target_id', ForeignKey('target.id'))

      @declared_attr
      def target(cls):
          return relationship(Target,
              primaryjoin=Target.id==cls.target_id   # это *неправильно*
          )
```

Сопоставив класс с помощью приведенного выше миксина, мы получим ошибку вида:

```bash
sqlalchemy.exc.InvalidRequestError: this ForeignKey's parent column is not
yet associated with a Table.
```

Это связано с тем, что столбец `target_id`, который мы вызвали в нашем методе `target()`, не является тем же столбцом, который декларативный фактически собирается сопоставить с нашей таблицей.

Условие выше разрешается с помощью лямбда:

```python
@declarative_mixin
class RefTargetMixin:
    @declared_attr
    def target_id(cls):
        return Column('target_id', ForeignKey('target.id'))

    @declared_attr
    def target(cls):
        return relationship(Target,
            primaryjoin=lambda: Target.id==cls.target_id
        )
```

или, альтернативно, строковая форма (которая в конечном итоге генерирует лямбда):

```python
@declarative_mixin
class RefTargetMixin:
    @declared_attr
    def target_id(cls):
        return Column('target_id', ForeignKey('target.id'))

    @declared_attr
    def target(cls):
        return relationship("Target",
            primaryjoin="Target.id==%s.target_id" % cls.__name__
        )
```

{% hint style="info" %}
Смотри также:

[Late-Evaluation of Relationship Arguments](https://docs.sqlalchemy.org/en/14/orm/basic\_relationships.html#orm-declarative-relationship-eval)
{% endhint %}

## Миксины с deferred(), column\_property() и других классах MapperProperty

Как и [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), все подклассы [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty), такие как [deferred()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.deferred), [column\_property()](https://docs.sqlalchemy.org/en/14/orm/mapping\_columns.html#sqlalchemy.orm.column\_property) и т. д., в конечном счете включают ссылки на столбцы, и поэтому при использовании с декларативными миксинами имеют требование [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr), чтобы не полагаться на копирование:

```python
@declarative_mixin
class SomethingMixin:

    @declared_attr
    def dprop(cls):
        return deferred(Column(Integer))

class Something(SomethingMixin, Base):
    __tablename__ = "something"
```

[column\_property()](https://docs.sqlalchemy.org/en/14/orm/mapping\_columns.html#sqlalchemy.orm.column\_property) или другая конструкция может ссылаться на другие столбцы миксина. Они копируются заранее, до того, как будет вызван [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr):

```python
@declarative_mixin
class SomethingMixin:
    x = Column(Integer)

    y = Column(Integer)

    @declared_attr
    def x_plus_y(cls):
        return column_property(cls.x + cls.y)
```

{% hint style="warning" %}
**Изменено в версии 1.0.0**: столбцы примеси копируются в окончательный сопоставленный класс, чтобы методы [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr) могли получить доступ к фактическому столбцу, который будет сопоставлен.
{% endhint %}

## Миксины прокси ассоциации и других атрибутов

Миксины могут указывать определяемые пользователем атрибуты, а также другие модули расширения, такие как [association\_proxy()](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html#sqlalchemy.ext.associationproxy.association\_proxy). Использование [declaration\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr) требуется в тех случаях, когда атрибут должен быть адаптирован специально для целевого подкласса. Примером может служить создание нескольких атрибутов [association\_proxy()](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html#sqlalchemy.ext.associationproxy.association\_proxy), каждый из которых нацелен на другой тип дочернего объекта. Ниже приведен пример миксина [association\_proxy()](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html#sqlalchemy.ext.associationproxy.association\_proxy), который предоставляет скалярный список строковых значений для реализующего класса:

```python
from sqlalchemy import Column, Integer, ForeignKey, String
from sqlalchemy.ext.associationproxy import association_proxy
from sqlalchemy.orm import declarative_base
from sqlalchemy.orm import declarative_mixin
from sqlalchemy.orm import declared_attr
from sqlalchemy.orm import relationship

Base = declarative_base()

@declarative_mixin
class HasStringCollection:
    @declared_attr
    def _strings(cls):
        class StringAttribute(Base):
            __tablename__ = cls.string_table_name
            id = Column(Integer, primary_key=True)
            value = Column(String(50), nullable=False)
            parent_id = Column(Integer,
                            ForeignKey('%s.id' % cls.__tablename__),
                            nullable=False)
            def __init__(self, value):
                self.value = value

        return relationship(StringAttribute)

    @declared_attr
    def strings(cls):
        return association_proxy('_strings', 'value')

class TypeA(HasStringCollection, Base):
    __tablename__ = 'type_a'
    string_table_name = 'type_a_strings'
    id = Column(Integer(), primary_key=True)

class TypeB(HasStringCollection, Base):
    __tablename__ = 'type_b'
    string_table_name = 'type_b_strings'
    id = Column(Integer(), primary_key=True)
```

Выше миксин **HasStringCollection** создает [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), которое ссылается на вновь сгенерированный класс с именем **StringAttribute**. Класс **StringAttribute** генерируется с собственным определением таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), которое является локальным для родительского класса с использованием миксина **HasStringCollection**. Он также создает объект [association\_proxy()](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html#sqlalchemy.ext.associationproxy.association\_proxy), который проксирует ссылки на атрибут `strings` на атрибут `value` каждого экземпляра **StringAttribute**.

**TypeA** или **TypeB** могут быть созданы, передавая конструктору аргумента `strings`, списка строк:

```python
ta = TypeA(strings=['foo', 'bar'])
tb = TypeB(strings=['bat', 'bar'])
```

Этот список сгенерирует коллекцию объектов **StringAttribute**, которые сохраняются в таблице, которая является локальной либо для таблицы `type_a_strings`, либо для таблицы `type_b_strings`:

```python
>>> print(ta._strings)
[<__main__.StringAttribute object at 0x10151cd90>,
    <__main__.StringAttribute object at 0x10151ce10>]
```

При построении [association\_proxy()](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html#sqlalchemy.ext.associationproxy.association\_proxy) необходимо использовать декоратор [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr), чтобы для каждого из классов **TypeA** и **TypeB** создавался отдельный объект [association\_proxy()](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html#sqlalchemy.ext.associationproxy.association\_proxy).

## Управление наследованием таблиц с помощью примесей

Атрибут **\_\_tablename\_\_** может использоваться для предоставления функции, которая будет определять имя таблицы, используемой для каждого класса в иерархии наследования, а также наличие у класса собственной отдельной таблицы.

Это достигается с помощью индикатора [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr) в сочетании с методом __ **\_\_tablename\_\_()**. Declarative всегда будет вызывать функцию [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr) для специальных имен __ **\_\_tablename\_\_**, **\_\_mapper\_args\_\_** и **\_\_table\_args\_\_** для **каждого сопоставленного класса в иерархии, за исключением случаев, когда они переопределены в подклассе**. Поэтому функция должна рассчитывать на получение каждого класса по отдельности и предоставление правильного ответа для каждого.

Например, чтобы создать миксин, который дает каждому классу простое имя таблицы на основе имени класса:

```python
from sqlalchemy.orm import declarative_mixin
from sqlalchemy.orm import declared_attr

@declarative_mixin
class Tablename:
    @declared_attr
    def __tablename__(cls):
        return cls.__name__.lower()

class Person(Tablename, Base):
    id = Column(Integer, primary_key=True)
    discriminator = Column('type', String(50))
    __mapper_args__ = {'polymorphic_on': discriminator}

class Engineer(Person):
    __tablename__ = None
    __mapper_args__ = {'polymorphic_identity': 'engineer'}
    primary_language = Column(String(50))
```

В качестве альтернативы мы можем изменить нашу функцию **\_\_tablename\_\_**, чтобы она возвращала `None` для подклассов, используя [has\_inherited\_table()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.has\_inherited\_table). Это приводит к тому, что эти подклассы сопоставляются с наследованием одной таблицы против родителя:

```python
from sqlalchemy.orm import declarative_mixin
from sqlalchemy.orm import declared_attr
from sqlalchemy.orm import has_inherited_table

@declarative_mixin
class Tablename:
    @declared_attr
    def __tablename__(cls):
        if has_inherited_table(cls):
            return None
        return cls.__name__.lower()

class Person(Tablename, Base):
    id = Column(Integer, primary_key=True)
    discriminator = Column('type', String(50))
    __mapper_args__ = {'polymorphic_on': discriminator}

class Engineer(Person):
    primary_language = Column(String(50))
    __mapper_args__ = {'polymorphic_identity': 'engineer'}
```

## &#x20;Миксины столбцов в сценариях наследования

В отличие от того, как обрабатываются **\_\_tablename\_\_** и другие специальные имена при использовании с [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr), когда мы смешиваем столбцы и свойства (например, отношения, свойства столбцов и т. д.), функция вызывается для **базового класса только** в иерархии. Ниже только класс **Person** получит столбец с именем **id**; сопоставление не будет выполнено для **Engineer**, которому не предоставлен первичный ключ:

```python
@declarative_mixin
class HasId:
    @declared_attr
    def id(cls):
        return Column('id', Integer, primary_key=True)

class Person(HasId, Base):
    __tablename__ = 'person'
    discriminator = Column('type', String(50))
    __mapper_args__ = {'polymorphic_on': discriminator}

class Engineer(Person):
    __tablename__ = 'engineer'
    primary_language = Column(String(50))
    __mapper_args__ = {'polymorphic_identity': 'engineer'}
```

Обычно при наследовании объединенных таблиц нам нужны столбцы с четкими именами в каждом подклассе. Однако в этом случае мы можем захотеть иметь столбец идентификатора **id** в каждой таблице и сделать так, чтобы они ссылались друг на друга через внешний ключ. Мы можем добиться этого в виде примеси, используя модификатор `declare_attr.cascading`, который указывает, что функция должна вызываться **для каждого класса в иерархии,** почти (см. предупреждение ниже) так же, как это делается для **\_\_tablename\_\_**:

```python
@declarative_mixin
class HasIdMixin:
    @declared_attr.cascading
    def id(cls):
        if has_inherited_table(cls):
            return Column(ForeignKey('person.id'), primary_key=True)
        else:
            return Column(Integer, primary_key=True)

class Person(HasIdMixin, Base):
    __tablename__ = 'person'
    discriminator = Column('type', String(50))
    __mapper_args__ = {'polymorphic_on': discriminator}

class Engineer(Person):
    __tablename__ = 'engineer'
    primary_language = Column(String(50))
    __mapper_args__ = {'polymorphic_identity': 'engineer'}
```

{% hint style="danger" %}
В настоящее время функция `declared_attr.cascading` **не позволяет** подклассу переопределять атрибут другой функцией или значением. Это текущее ограничение в механике разрешения `@declared_attr`, и при обнаружении этого условия выдается предупреждение. Этого ограничения **не существует** для имен специальных атрибутов, таких как **\_\_tablename\_\_**, которые внутренне разрешаются иначе, чем в `declared_attr.cascading`.
{% endhint %}

{% hint style="info" %}
**Новое в версии 1.0.0**: добавлена `declared_attr.cascading`.
{% endhint %}

## Объединение аргументов Table/Mapper из нескольких миксинов

В случае **\_\_table\_args\_\_** или **\_\_mapper\_args\_\_**, указанных с декларативными примесями, вы можете захотеть объединить некоторые параметры из нескольких примесей с теми, которые вы хотите определить в самом классе. Здесь можно использовать декоратор [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr) для создания определяемых пользователем подпрограмм сопоставления, которые извлекают данные из нескольких коллекций:

```python
from sqlalchemy.orm import declarative_mixin
from sqlalchemy.orm import declared_attr

@declarative_mixin
class MySQLSettings:
    __table_args__ = {'mysql_engine':'InnoDB'}

@declarative_mixin
class MyOtherMixin:
    __table_args__ = {'info':'foo'}

class MyModel(MySQLSettings, MyOtherMixin, Base):
    __tablename__='my_model'

    @declared_attr
    def __table_args__(cls):
        args = dict()
        args.update(MySQLSettings.__table_args__)
        args.update(MyOtherMixin.__table_args__)
        return args

    id =  Column(Integer, primary_key=True)
```

## Создание индексов с помощью миксинов

Чтобы определить именованный, потенциально многоколоночный индекс, который применяется ко всем таблицам, полученным из миксина, используйте «встроенную» форму индекса [Index](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.Index) и установите его как часть **\_\_table\_args\_\_**:

```python
@declarative_mixin
class MyMixin:
    a =  Column(Integer)
    b =  Column(Integer)

    @declared_attr
    def __table_args__(cls):
        return (Index('test_idx_%s' % cls.__tablename__, 'a', 'b'),)

class MyModel(MyMixin, Base):
    __tablename__ = 'atable'
    c =  Column(Integer,primary_key=True)
```
