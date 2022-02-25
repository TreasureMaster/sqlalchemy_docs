# Внутренние методы коллекций

Различные внутренние методы.

| Название объекта                                                                                                                                                        | Описание                                                                                                                                              |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| [bulk\_replace](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.bulk\_replace)(values, existing\_adapter, new\_adapter\[, initiator]) | Загрузите новую коллекцию, запуская события на основе предыдущего членства.                                                                           |
| [collection](https://docs.sqlalchemy.org/en/14/orm/collections.html#id0)                                                                                                | Декораторы для классов коллекций сущностей.                                                                                                           |
| [collection\_adapter](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.collection\_adapter)                                            | Получить [CollectionAdapter](vnutrennie-metody-kollekcii.md#class-sqlalchemy.orm.collections.collectionadapter-attr-owner\_state-data) для коллекции. |
| [CollectionAdapter](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.CollectionAdapter)                                                | Мосты между ORM и произвольными коллекциями Python.                                                                                                   |
| [InstrumentedDict](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.InstrumentedDict)                                                  | Инструментированная версия встроенного словаря.                                                                                                       |
| [InstrumentedList](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.InstrumentedList)                                                  | Инструментированная версия встроенного списка.                                                                                                        |
| [InstrumentedSet](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.InstrumentedSet)                                                    | Инструментальная версия встроенного множества.                                                                                                        |
| [prepare\_instrumentation](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.prepare\_instrumentation)(factory)                         | Подготовьте вызываемый объект для будущего использования в качестве фабрики классов коллекций.                                                        |

### _function_ sqlalchemy.orm.collections.bulk\_replace(_values_, _existing\_adapter_, _new\_adapter_, _initiator=None_)

Загружает новую коллекцию, запуская события на основе предыдущего членства.

Добавляет экземпляры значений **values** в **new\_adapter**. События будут запускаться для любого экземпляра, отсутствующего в файле **existing\_adapter**. Для любых экземпляров в **existing\_adapter**, отсутствующих в значениях **values**, будут запущены события удаления.

#### Параметры:

* **values** - Итерируемый объект экземпляров членов коллекции
* **existing\_adapter** - [CollectionAdapter](vnutrennie-metody-kollekcii.md#class-sqlalchemy.orm.collections.collectionadapter-attr-owner\_state-data) экземпляров, подлежащих замене
* **new\_adapter** - Пустой [CollectionAdapter](vnutrennie-metody-kollekcii.md#class-sqlalchemy.orm.collections.collectionadapter-attr-owner\_state-data) для загрузки значений **values**

### _class_ sqlalchemy.orm.collections.collection

Декораторы для классов коллекций сущностей.

Декораторы делятся на две группы: аннотации и рецепты перехвата.

Аннотирующие декораторы (**appender**, **remover**, **iterator**, **convert**, **internally\_instrumented**) указывают цель метода и не принимают аргументов. Они не пишутся со скобками:

```python
@collection.appender
def append(self, append): ...
```

Все декораторы рецептов требуют круглых скобок, даже те, которые не принимают аргументов:

```python
@collection.adds('entity')
def insert(self, position, entity): ...

@collection.removes_return()
def popitem(self): ...
```

### sqlalchemy.orm.collections.collection\_adapter _= operator.attrgetter('\_sa\_adapter')_

Получить [CollectionAdapter](vnutrennie-metody-kollekcii.md#class-sqlalchemy.orm.collections.collectionadapter-attr-owner\_state-data) для коллекции.

### _class_ sqlalchemy.orm.collections.CollectionAdapter(_attr_, _owner\_state_, _data_)

Мосты между ORM и произвольными коллекциями Python.

Проксирует операции коллекции базового уровня (**append**, **remove**, **iterate**) в базовую коллекцию Python и генерирует события **add**/**remove** для сущностей, входящих в коллекцию или покидающих ее.

ORM использует [CollectionAdapter](vnutrennie-metody-kollekcii.md#class-sqlalchemy.orm.collections.collectionadapter-attr-owner\_state-data) исключительно для взаимодействия с коллекциями сущностей.

### _class_ sqlalchemy.orm.collections.InstrumentedDict

Инструментированная версия встроенного **dict**.

{% hint style="info" %}
Сигнатура класса:

class `sqlalchemy.orm.collections.InstrumentedDict` (`builtins.dict`)
{% endhint %}

### _class_ sqlalchemy.orm.collections.InstrumentedList(_iterable=()_, _/_)

Инструментированная версия встроенного списка.

{% hint style="info" %}
Сигнатура класса:

class `sqlalchemy.orm.collections.InstrumentedList` (`builtins.list`)
{% endhint %}

### _class_ sqlalchemy.orm.collections.InstrumentedSet

Инструментальная версия встроенного множества.

{% hint style="info" %}
Сигнатура класса:

class `sqlalchemy.orm.collections.InstrumentedSet` (`builtins.set`)
{% endhint %}

### _function_ sqlalchemy.orm.collections.prepare\_instrumentation(_factory_)

Подготавливает вызываемый объект для будущего использования в качестве фабрики классов коллекций.

Учитывая фабрику класса коллекции (вызываемую либо по типу, либо без аргументов), возвращает другую фабрику, которая будет создавать совместимые экземпляры при вызове.

Эта функция отвечает за преобразование **collection\_class=list** в поведение во время выполнения **collection\_class=InstrumentedList**.
