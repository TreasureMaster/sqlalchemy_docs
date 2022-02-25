# class Mapper

## _class_ sqlalchemy.orm.Mapper(_class\__, _local\_table=None_, _properties=None_, _primary\_key=None_, _non\_primary=False_, _inherits=None_, _inherit\_condition=None_, _inherit\_foreign\_keys=None_, _always\_refresh=False_, _version\_id\_col=None_, _version\_id\_generator=None_, _polymorphic\_on=None_, _\_polymorphic\_map=None_, _polymorphic\_identity=None_, _concrete=False_, _with\_polymorphic=None_, _polymorphic\_load=None_, _allow\_partial\_pks=True_, _batch=True_, _column\_prefix=None_, _include\_properties=None_, _exclude\_properties=None_, _passive\_updates=True_, _passive\_deletes=False_, _confirm\_deleted\_rows=True_, _eager\_defaults=False_, _legacy\_is\_orphan=False_, _\_compiled\_cache\_size=100_)

Определяет связь между классом Python и таблицей базы данных или другой реляционной структурой, чтобы можно было выполнять операции ORM с классом.

Объект **Mapper** создается с использованием методов сопоставления, присутствующих в объекте [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl). Сведения о создании экземпляров новых объектов **Mapper** см. в разделе [Сопоставление классов Python](../otobrazhenie-klassov-python.md).

{% hint style="info" %}
Сигнатура класса

class [`sqlalchemy.orm.Mapper`](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) (`sqlalchemy.orm.ORMFromClauseRole`, `sqlalchemy.orm.ORMEntityColumnsClauseRole`, `sqlalchemy.sql.traversals.MemoizedHasCacheKey`, [`sqlalchemy.orm.base.InspectionAttr`](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.InspectionAttr))
{% endhint %}

### _method_ sqlalchemy.orm.Mapper.\_\_init\_\_(_class\__, _local\_table=None_, _properties=None_, _primary\_key=None_, _non\_primary=False_, _inherits=None_, _inherit\_condition=None_, _inherit\_foreign\_keys=None_, _always\_refresh=False_, _version\_id\_col=None_, _version\_id\_generator=None_, _polymorphic\_on=None_, _\_polymorphic\_map=None_, _polymorphic\_identity=None_, _concrete=False_, _with\_polymorphic=None_, _polymorphic\_load=None_, _allow\_partial\_pks=True_, _batch=True_, _column\_prefix=None_, _include\_properties=None_, _exclude\_properties=None_, _passive\_updates=True_, _passive\_deletes=False_, _confirm\_deleted\_rows=True_, _eager\_defaults=False_, _legacy\_is\_orphan=False_, _\_compiled\_cache\_size=100_)



Создайте новый объект [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

Этот конструктор отражается как общедоступная функция API; см. [sqlalchemy.orm.mapper()](func-mapper.md#function-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary) для полного описания использования и аргументов.

### _method_ sqlalchemy.orm.Mapper.add\_properties(_dict\_of\_properties_)

Добавляет данный словарь свойств в этот преобразователь, используя **add\_property**.

### _method_ sqlalchemy.orm.Mapper.add\_property(_key_, _prop_)

Добавляет отдельное свойство **MapperProperty** в этот преобразователь.

Если сопоставитель еще не настроен, просто добавьте свойство в исходный словарь свойств, отправленный в конструктор. Если этот **Mapper** уже настроен, то данное **MapperProperty** настраивается немедленно.

### _attribute_ sqlalchemy.orm.Mapper.all\_orm\_descriptors

Пространство имен всех атрибутов [InspectionAttr](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.InspectionAttr), связанных с сопоставленным классом.

Эти атрибуты во всех случаях являются [дескрипторами](https://docs.sqlalchemy.org/en/14/glossary.html#term-descriptors) Python, связанными с отображаемым классом или его суперклассами.

Это пространство имен включает атрибуты, сопоставленные с классом, а также атрибуты, объявленные модулями расширения. Он включает в себя любой тип дескриптора Python, наследуемый от [InspectionAttr](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.InspectionAttr). Сюда входят [QueryableAttribute](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.QueryableAttribute), а также типы расширений, такие как [hybrid\_property](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html#sqlalchemy.ext.hybrid.hybrid\_property), [hybrid\_method](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html#sqlalchemy.ext.hybrid.hybrid\_method) и [AssociationProxy](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html#sqlalchemy.ext.associationproxy.AssociationProxy).

Чтобы различать сопоставленные атрибуты и атрибуты расширения, атрибут [InspectionAttr.extension\_type](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.InspectionAttr.extension\_type) будет ссылаться на константу, которая различает разные типы расширений.

Сортировка атрибутов основана на следующих правилах:

1. Итерация по классу и его суперклассам в порядке от подкласса к суперклассу (т.е. итерация по **cls.\_\_mro\_\_**)
2. Для каждого класса выдайте атрибуты в том порядке, в котором они появляются в **\_\_dict\_\_**, за исключением тех, которые указаны на шаге 3 ниже. В Python 3.6 и более поздних версиях этот порядок будет таким же, как и при построении класса, за исключением атрибутов, которые были добавлены приложением или mapper постфактум.
3. Если определенный ключ атрибута также находится в суперклассе **\_\_dict\_\_**, то он включается в итерацию для этого класса, а не для класса, в котором он впервые появился.

Описанный выше процесс создает порядок, который является детерминированным с точки зрения порядка, в котором атрибуты были присвоены классу.

{% hint style="warning" %}
**Изменено в версии 1.3.19**: обеспечен детерминированный порядок для [Mapper.all\_orm\_descriptors](class-mapper.md#attribute-sqlalchemy.orm.mapper.all\_orm\_descriptors).
{% endhint %}

При работе с [QueryableAttribute](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.QueryableAttribute) атрибут [QueryableAttribute.property](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.QueryableAttribute.property) ссылается на свойство [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty), которое вы получаете, обращаясь к набору сопоставленных свойств через [Mapper.attrs](class-mapper.md#attribute-sqlalchemy.orm.mapper.attrs).

{% hint style="danger" %}
Пространство имен аксессора [Mapper.all\_orm\_descriptors](class-mapper.md#attribute-sqlalchemy.orm.mapper.all\_orm\_descriptors) является экземпляром **OrderedProperties**. Это объект, похожий на словарь, который включает в себя небольшое количество именованных методов, таких как **OrderedProperties.items()** и **OrderedProperties.values()**. При динамическом доступе к атрибутам отдавайте предпочтение схеме dict-access, например, `mapper.all_orm_descriptors[somename]` вместо `getattr(mapper.all_orm_descriptors, somename)`, чтобы избежать конфликтов имен.
{% endhint %}

{% hint style="info" %}
Смотри также:

``[`Mapper.attrs`](class-mapper.md#attribute-sqlalchemy.orm.mapper.attrs)``
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.attrs

Пространство имен всех объектов [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty), связанных с этим преобразователем.

Это объект, который предоставляет каждое свойство на основе его имени ключа. Например, преобразователь для класса **User**, который имеет атрибут **User.name**, предоставит **mapper.attrs.name**, который будет [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty), представляющим столбец **name**. Объект пространства имен также можно итерировать, что приведет к получению каждого свойства [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty).

[Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal) имеет несколько предварительно отфильтрованных представлений этого атрибута, которые ограничивают типы возвращаемых свойств, включая synonyms, column\_attrs, relationships и composites.

{% hint style="danger" %}
Пространство имен аксессора [Mapper.attrs](class-mapper.md#attribute-sqlalchemy.orm.mapper.attrs) является экземпляром **OrderedProperties**. Это объект, похожий на словарь, который включает в себя небольшое количество именованных методов, таких как **OrderedProperties.items()** и **OrderedProperties.values()**. При динамическом доступе к атрибутам отдавайте предпочтение схеме dict-access, например, `mapper.attrs[somename]` вместо `getattr(mapper.attrs, somename)`, чтобы избежать конфликтов имен.
{% endhint %}

{% hint style="info" %}
Смотри также:

``[`Mapper.all_orm_descriptors`](class-mapper.md#attribute-sqlalchemy.orm.mapper.all\_orm\_descriptors)``
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.base\_mapper _= None_

Самый базовый [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal) в цепочке наследования.

В сценарии без наследования этим атрибутом всегда будет этот [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal). В сценарии наследования он ссылается на **Mapper**, который является родительским для всех других объектов **Mapper** в цепочке наследования.

Это атрибут только для чтения, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _attribute_ sqlalchemy.orm.Mapper.c _= None_

Синоним [Mapper.columns](class-mapper.md#attribute-sqlalchemy.orm.mapper.columns-none).

### _method_ sqlalchemy.orm.Mapper.cascade\_iterator(_type\__, _state_, _halt\_on=None_)

Итерирует каждый элемент и его преобразователь (mapper) в графе объектов для всех отношений, которые соответствуют заданному каскадному правилу.

#### Параметры:

* **type\_** - Название каскадного правила (например, `«save-update»`, `«delete»` и т. д.).

{% hint style="info" %}
каскад `"all"` здесь не принимается. Универсальную функцию обхода объекта см. в разделе Как обойти все объекты, связанные с данным объектом? ([How do I walk all objects that are related to a given object?](https://docs.sqlalchemy.org/en/14/faq/sessions.html#faq-walk-objects))
{% endhint %}

* **state** - Ведущий **InstanceState**. Дочерние элементы будут обрабатываться в соответствии с отношениями, определенными для сопоставления этого объекта.

#### Возвращает:

* метод дает (yield) отдельные экземпляры объекта.

{% hint style="info" %}
Смотри также:

[Cascades](https://docs.sqlalchemy.org/en/14/orm/cascades.html#unitofwork-cascades)

[How do I walk all objects that are related to a given object?](https://docs.sqlalchemy.org/en/14/faq/sessions.html#faq-walk-objects) - иллюстрирует общую функцию для обхода всех объектов, не полагаясь на каскады.
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.class\_ _= None_

Класс Python, который отображает этот [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

Это атрибут только для чтения, определяемый во время создания _mapper_. Поведение не определено, если оно изменено напрямую.

### _attribute_ sqlalchemy.orm.Mapper.class\_manager _= None_

[ClassManager](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ClassManager), который поддерживает прослушиватели событий и дескрипторы, привязанные к классу, для этого [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

Это атрибут только для чтения, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _attribute_ sqlalchemy.orm.Mapper.column\_attrs

Возвращает пространство имен всех свойств [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty), поддерживаемых этим [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

{% hint style="info" %}
Смотри также:

``[`Mapper.attrs`](class-mapper.md#attribute-sqlalchemy.orm.mapper.attrs) - пространство имен всех объектов [`MapperProperty`](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty).
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.columns _= None_

Набор объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) или других скалярных выражений, поддерживаемых этим [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

Коллекция ведет себя так же, как атрибут **c** в любом объекте [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), за исключением того, что присутствуют только те столбцы, которые включены в это сопоставление, и ключи на основе имени атрибута, определенного в сопоставлении, не обязательно ключевого атрибута **key** самого столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column). Кроме того, здесь также присутствуют скалярные выражения, отображаемые с помощью [column\_property()](../otobrazhenie-kolonok-i-vyrazhenii/otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs).

Это атрибут только для чтения, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _method_ sqlalchemy.orm.Mapper.common\_parent(_other_)

Возвращает `True`, если данный преобразователь (mapper) имеет общего унаследованного родителя как этот преобразователь (mapper).

### _attribute_ sqlalchemy.orm.Mapper.composites

Возвращает пространство имен всех свойств [CompositeProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.CompositeProperty), поддерживаемых этим [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

{% hint style="info" %}
Смотри также:

[Mapper.attrs](class-mapper.md#attribute-sqlalchemy.orm.mapper.attrs) — пространство имен всех объектов [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty).
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.concrete _= None_

Представляет `True`, если этот преобразователь (mapper) является конкретным преобразователем наследования.

Это атрибут только для чтения, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _attribute_ sqlalchemy.orm.Mapper.configured _= False_

Представляет `True`, если этот [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal) был настроен.

Это атрибут только для чтения, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

{% hint style="info" %}
Смотри также:

``[`configure_mappers()`](func-object\_mapper-identity\_key-etc..md#function-sqlalchemy.orm.configure\_mappers).
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.entity

Часть инспекционного API.

Возвращает **self.class\_**.

### _method_ sqlalchemy.orm.Mapper.get\_property(_key_, _\_configure\_mappers=True_)

Возвращает **MapperProperty**, связанное с данным ключом.

### _method_ sqlalchemy.orm.Mapper.get\_property\_by\_column(_column_)

Передает объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), возвращая [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty), которое сопоставлено с этим столбцом.

### _method_ sqlalchemy.orm.Mapper.identity\_key\_from\_instance(_instance_)

Возвращает ключ идентификации для данного экземпляра на основе его атрибутов первичного ключа.

Если состояние экземпляра истекло, вызов этого метода приведет к проверке базы данных, чтобы увидеть, был ли объект удален. Если строка больше не существует, возникает ошибка [ObjectDeletedError](https://docs.sqlalchemy.org/en/14/orm/exceptions.html#sqlalchemy.orm.exc.ObjectDeletedError).

Это значение обычно также находится в состоянии экземпляра под именем атрибута _**key**_.

### _method_ sqlalchemy.orm.Mapper.identity\_key\_from\_primary\_key(_primary\_key_, _identity\_token=None_)

Возвращает ключ карты идентификации для использования при сохранении/извлечении элемента из карты идентификации.

#### Параметры:

* **primary\_key** - Список значений, указывающих идентификатор.

### _method_ sqlalchemy.orm.Mapper.identity\_key\_from\_row(_row_, _identity\_token=None_, _adapter=None_)

Возвращает ключ карты идентификации для использования при сохранении/извлечении элемента из карты идентификации.

#### Параметры:

* **row** - Экземпляр строки [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row). Столбцы, отображаемые этим [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal), должны обнаруживаться в строке, предпочтительно напрямую через объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) (как в случае, когда выполняется конструкция [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select)) или через строковые имена вида `<tablename>_<colname>`.

### _attribute_ sqlalchemy.orm.Mapper.inherits _= None_

Ссылается на [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal), от которого наследуется этот **Mapper**, если таковой имеется.

Это атрибут только для чтения, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _attribute_ sqlalchemy.orm.Mapper.is\_mapper _= True_

Часть инспекционного API.

### _method_ sqlalchemy.orm.Mapper.is\_sibling(_other_)

Возвращает `True`, если другой преобразователь (mapper) является родственным братом этого. Общий родитель, но другая ветвь

### _method_ sqlalchemy.orm.Mapper.isa(_other_)

Возвращает `True`, если этот преобразователь (mapper) наследуется от данного преобразователя.

### _attribute_ sqlalchemy.orm.Mapper.iterate\_properties

Возвращает итератор всех объектов **MapperProperty**.

### _attribute_ sqlalchemy.orm.Mapper.local\_table _= None_

Объект [Selectable](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Selectable), которым управляет этот [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

Обычно это экземпляр [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) или [Alias](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Alias). Также может быть `None`.

**«local»** таблица является выбираемой (selectable), за управление которой [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal) непосредственно отвечает с точки зрения доступа к атрибутам и сброса. Для ненаследующих преобразователей локальная таблица совпадает с **«mapped»** таблицей. Для сопоставителей наследования объединенных таблиц **local\_table** будет конкретной подтаблицей общего **«join»**, которое представляет этот сопоставитель. Если этот преобразователь является наследующим преобразователем одной таблицы, **local\_table** будет `None`.

{% hint style="info" %}
Смотри также:

``[`Mapper.persist_selectable`](class-mapper.md#attribute-sqlalchemy.orm.mapper.persist\_selectable-none).
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.mapped\_table

{% hint style="warning" %}
**Устарело с версии 1.3**: используйте **`.persist_selectable`**
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.mapper

Часть инспекционного API.

Возвращает самого себя (self).

### _attribute_ sqlalchemy.orm.Mapper.non\_primary _= None_

Представляет `True`, если этот преобразователь (mapper) является «неосновным» преобразователем, например, сопоставитель (mapper), который используется только для выбора строк, но не для управления сохранением.

Это атрибут только для чтения, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _attribute_ sqlalchemy.orm.Mapper.persist\_selectable _= None_

Объект [Selectable](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Selectable), на который сопоставляется этот [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

Обычно это экземпляр [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), [Join](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Join) или [Alias](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Alias).

**Mapper.persist\_selectable** отличается от [Mapper.selectable](class-mapper.md#attribute-sqlalchemy.orm.mapper.selectable) тем, что первый представляет столбцы, которые отображаются в этом классе или его суперклассах, тогда как последний может быть «полиморфным» выбираемым selectable, содержащим дополнительные столбцы, которые фактически отображаются только в подклассах.

«persist selectable» — это «вещь, на которую записывает mapper», а «selectable» — это «вещь, из которой mapper выбирает».

**Mapper.persist\_selectable** также отделен от [Mapper.local\_table](class-mapper.md#attribute-sqlalchemy.orm.mapper.local\_table-none), который представляет собой набор столбцов, локально сопоставленных непосредственно с этим классом.

{% hint style="info" %}
Смотри также:

``[`Mapper.selectable`](class-mapper.md#attribute-sqlalchemy.orm.mapper.selectable).

``[`Mapper.local_table`](class-mapper.md#attribute-sqlalchemy.orm.mapper.local\_table-none).
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.polymorphic\_identity _= None_

Представляет идентификатор, который сопоставляется со столбцом [Mapper.polymorphic\_on](class-mapper.md#attribute-sqlalchemy.orm.mapper.polymorphic\_on-none) во время загрузки строки результата.

Используемый только с наследованием, этот объект может быть любого типа, сравнимого с типом столбца, представленного [Mapper.polymorphic\_on](class-mapper.md#attribute-sqlalchemy.orm.mapper.polymorphic\_on-none).

Это атрибут только для чтения, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _method_ sqlalchemy.orm.Mapper.polymorphic\_iterator()

Итерирует коллекцию, включая этот преобразователь (mapper) и все потомки преобразователей.

Сюда входят не только непосредственно наследующие преобразователи, но и все их наследующие преобразователи.

Чтобы перебрать всю иерархию, используйте **mapper.base\_mapper.polymorphic\_iterator()**.

### _attribute_ sqlalchemy.orm.Mapper.polymorphic\_map _= None_

Сопоставление идентификаторов «полиморфной идентичности», сопоставленных с экземплярами [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal), в рамках сценария наследования.

Идентификаторы могут быть любого типа, сравнимого с типом столбца, представленного [Mapper.polymorphic\_on](class-mapper.md#attribute-sqlalchemy.orm.mapper.polymorphic\_on-none).

Цепочка наследования mappers будет ссылаться на один и тот же полиморфный объект карты. Объект используется для сопоставления входящих строк результатов с целевыми преобразователями.

Это атрибут _**только для чтения**_, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _attribute_ sqlalchemy.orm.Mapper.polymorphic\_on _= None_

Столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) или выражение SQL, указанное в качестве аргумента **polymorphic\_on** для этого [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal) в рамках сценария наследования.

Этот атрибут обычно является экземпляром столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), но также может быть выражением, например производным от [cast()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.cast).

Это атрибут _**только для чтения**_, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _attribute_ sqlalchemy.orm.Mapper.primary\_key _= None_

Итерируемый объект, содержащий коллекцию объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), которые составляют «первичный ключ» сопоставленной таблицы с точки зрения этого [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

Этот список противоречит выбираемому в [Mapper.persist\_selectable](class-mapper.md#attribute-sqlalchemy.orm.mapper.persist\_selectable-none). В случае наследования преобразователей некоторые столбцы могут управляться преобразователем суперкласса. Например, в случае соединения **JOIN** первичный ключ определяется всеми столбцами первичного ключа во всех таблицах, на которые ссылается соединение **JOIN**.

Список также не обязательно совпадает с коллекцией столбцов первичного ключа, связанной с базовыми таблицами; [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal) имеет аргумент **primary\_key**, который может переопределить то, что **Mapper** считает столбцами первичного ключа.

Это атрибут _**только для чтения**_, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _method_ sqlalchemy.orm.Mapper.primary\_key\_from\_instance(_instance_)

Возвращает список значений первичного ключа для данного экземпляра.

Если состояние экземпляра истекло, вызов этого метода приведет к проверке базы данных, чтобы увидеть, был ли объект удален. Если строка больше не существует, возникает ошибка [ObjectDeletedError](https://docs.sqlalchemy.org/en/14/orm/exceptions.html#sqlalchemy.orm.exc.ObjectDeletedError).

### _method_ sqlalchemy.orm.Mapper.primary\_mapper()

Возвращает первичный преобразователь (mapper), соответствующий ключу класса этого преобразователя (классу).

### _attribute_ sqlalchemy.orm.Mapper.relationships

Пространство имен всех свойств [RelationshipProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty), поддерживаемых этим [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

{% hint style="danger" %}
пространство имен аксессора **Mapper.relationships** является экземпляром **OrderedProperties**. Это объект, похожий на словарь, который включает в себя небольшое количество именованных методов, таких как **OrderedProperties.items()** и **OrderedProperties.values()**. При динамическом доступе к атрибутам отдавайте предпочтение схеме dict-access, например, `mapper.relationships[somename]` вместо `getattr(mapper.relationships, somename)`, чтобы избежать конфликтов имен.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Mapper.attrs](class-mapper.md#attribute-sqlalchemy.orm.mapper.attrs) — пространство имен всех объектов [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty).
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.selectable

Конструкция **FromClause**, которую этот [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal) выбирает по умолчанию.

Обычно это эквивалентно [persist\_selectable](class-mapper.md#attribute-sqlalchemy.orm.mapper.persist\_selectable-none), если только не используется функция **with\_polymorphic**, и в этом случае возвращается полный «полиморфный» selectable.

### _attribute_ sqlalchemy.orm.Mapper.self\_and\_descendants

Коллекция, включающая этот преобразователь (mapper) и все дочерние преобразователи.

Сюда входят не только непосредственно наследующие преобразователи, но и все их наследующие преобразователи.

### _attribute_ sqlalchemy.orm.Mapper.single _= None_

Предоставляет `True`, если этот преобразователь (mapper) является преобразователем наследования одной таблицы.

[Mapper.local\_table](class-mapper.md#attribute-sqlalchemy.orm.mapper.local\_table-none) будет `None`, если этот флаг установлен.

Это атрибут _**только для чтения**_, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _attribute_ sqlalchemy.orm.Mapper.synonyms

Возвращает пространство имен всех свойств [SynonymProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.SynonymProperty), поддерживаемых этим [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

{% hint style="info" %}
Смотри также:

[Mapper.attrs](class-mapper.md#attribute-sqlalchemy.orm.mapper.attrs) — пространство имен всех объектов [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty).
{% endhint %}

### _attribute_ sqlalchemy.orm.Mapper.tables _= None_

Итерируемый объект, содержащий коллекцию объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), о которых знает этот [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal).

Если преобразователь (mapper) сопоставляется с [Join](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Join) или [Alias](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Alias), представляющим [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), здесь будут представлены отдельные объекты таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), составляющие полную конструкцию.

Это атрибут _**только для чтения**_, определяемый во время создания mapper. Поведение не определено, если оно изменено напрямую.

### _attribute_ sqlalchemy.orm.Mapper.validators _= None_

Неизменяемый словарь атрибутов, оформленных с помощью декоратора [validates()](../otobrazhenie-kolonok-i-vyrazhenii/izmenenie-povedeniya-atributov.md#function-sqlalchemy.orm.validates-names-kw).

Словарь содержит имена строковых атрибутов в виде ключей, сопоставленных с фактическим методом проверки.

### _attribute_ sqlalchemy.orm.Mapper.with\_polymorphic\_mappers

Список объектов [Mapper](class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal), включенных в «полиморфный» запрос по умолчанию.
