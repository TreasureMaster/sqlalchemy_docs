# Отображение колонок таблицы

Поведение [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper) по умолчанию состоит в том, чтобы собрать все столбцы в сопоставленной таблице [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) в атрибуты сопоставленного объекта, каждый из которых назван в соответствии с именем самого столбца (в частности, ключевого атрибута `key` столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table)). Это поведение можно изменить несколькими способами.

## Именование столбцов, отличное от имен атрибутов

Сопоставление по умолчанию использует то же имя для столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), что и сопоставленный атрибут, в частности, оно соответствует атрибуту `Column.key` в столбце, который по умолчанию совпадает с `Column.name`.

Имя, назначенное атрибуту Python, который сопоставляется с [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), может отличаться как от `Column.name`, так и от `Column.key`, просто назначая его таким образом, как мы иллюстрируем здесь в декларативном сопоставлении:

```python
class User(Base):
    __tablename__ = 'user'
    id = Column('user_id', Integer, primary_key=True)
    name = Column('user_name', String(50))
```

Где выше `User.id` разрешается в столбец с именем **user\_id**, а `User.name` разрешается в столбец с именем **user\_name**.

При сопоставлении с существующей таблицей на объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) можно ссылаться напрямую:

```python
class User(Base):
    __table__ = user_table
    id = user_table.c.user_id
    name = user_table.c.user_name
```

Соответствующая техника для императивного сопоставления заключается в помещении нужного ключа в словарь [mapper.properties](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.properties) с нужным ключом:

```python
mapper_registry.map_imperatively(User, user_table, properties={
   'id': user_table.c.user_id,
   'name': user_table.c.user_name,
})
```

В следующем разделе мы более подробно рассмотрим использование `.key`.

## Автоматизация схем именования столбцов из отраженных таблиц

В предыдущем разделе [Именование столбцов, отличное от имен атрибутов](otobrazhenie-kolonok-tablicy.md#imenovanie-stolbcov-v-otlichie-ot-imen-atributov), мы показали, как столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), явно сопоставленный с классом, может иметь имя атрибута, отличное от имени столбца. Но что, если мы не перечисляем объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) явно, а вместо этого автоматизируем создание объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) с помощью отражения (например, как описано в разделе **Отражение объектов базы данных**)? В этом случае мы можем использовать событие [DDLEvents.column\_reflect()](https://docs.sqlalchemy.org/en/14/core/events.html#sqlalchemy.events.DDLEvents.column\_reflect) для перехвата создания объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) и предоставления им `Column.key` по нашему выбору. Событие легче всего связать с используемым объектом [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), например, ниже мы используем тот, который связан с экземпляром [declarative\_base](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base):

```python
@event.listens_for(Base.metadata, "column_reflect")
def column_reflect(inspector, table, column_info):
    # установить column.key = "attr_<lower_case_name>"
    column_info['key'] = "attr_%s" % column_info['name'].lower()
```

С указанным выше событием отражение объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) будет перехвачено нашим событием, которое добавляет новый элемент `«.key»`, например, в отображении, как показано ниже:

```python
class MyClass(Base):
    __table__ = Table("some_table", Base.metadata,
                autoload_with=some_engine)
```

Этот подход также работает с расширением [Automap](https://docs.sqlalchemy.org/en/14/orm/extensions/automap.html). См. раздел «**Перехват определений столбцов**» для получения дополнительной информации.

{% hint style="info" %}
Смотри также:

[`DDLEvents.column_reflect()`](https://docs.sqlalchemy.org/en/14/core/events.html#sqlalchemy.events.DDLEvents.column\_reflect)

[Intercepting Column Definitions](https://docs.sqlalchemy.org/en/14/orm/extensions/automap.html#automap-intercepting-columns) - `в документации` [Automap](https://docs.sqlalchemy.org/en/14/orm/extensions/automap.html)
{% endhint %}

## Именование всех столбцов с префиксом

Быстрый подход к именам столбцов с префиксом, обычно при сопоставлении с существующим объектом [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), заключается в использовании `column_prefix`:

```python
class User(Base):
    __table__ = user_table
    __mapper_args__ = {'column_prefix':'_'}
```

Приведенное выше поместит имена атрибутов, такие как `_user_id`, `_user_name`, `_password` и т. д., в сопоставленный класс пользователя.

Этот подход редко используется в современном использовании. Для работы с отраженными таблицами более гибким подходом является использование описанного в разделе [Автоматизация схем именования столбцов из отраженных таблиц](otobrazhenie-kolonok-tablicy.md#avtomatizaciya-skhem-imenovaniya-stolbcov-iz-otrazhennykh-tablic).

## Использование column\_property для параметров уровня столбца

Параметры могут быть указаны при сопоставлении столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) с помощью функции [column\_property()](https://docs.sqlalchemy.org/en/14/orm/mapping\_columns.html#sqlalchemy.orm.column\_property). Эта функция явно создает свойство [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty), используемое функцией [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper) для отслеживания столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column); обычно [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper) создает это автоматически. Используя [column\_property()](https://docs.sqlalchemy.org/en/14/orm/mapping\_columns.html#sqlalchemy.orm.column\_property), мы можем передать дополнительные аргументы о том, как мы хотим, чтобы столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) отображался. Ниже мы передаем параметр `active_history`, который указывает, что изменение значения этого столбца должно привести к загрузке прежнего значения в первую очередь:

```python
from sqlalchemy.orm import column_property

class User(Base):
    __tablename__ = 'user'

    id = Column(Integer, primary_key=True)
    name = column_property(Column(String(50)), active_history=True)
```

[column\_property()](https://docs.sqlalchemy.org/en/14/orm/mapping\_columns.html#sqlalchemy.orm.column\_property) также используется для сопоставления одного атрибута с несколькими столбцами. Этот вариант использования возникает при сопоставлении с [join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.join), атрибуты которого приравниваются друг к другу:

```python
class User(Base):
    __table__ = user.join(address)

    # присоединяет "user.id", "address.user_id"
    # к атрибуту "id"
    id = column_property(user_table.c.id, address_table.c.user_id)
```

Дополнительные примеры такого использования см. в разделе **Сопоставление класса с несколькими таблицами**.

Другое место, где необходимо использовать [column\_property()](https://docs.sqlalchemy.org/en/14/orm/mapping\_columns.html#sqlalchemy.orm.column\_property), — указать выражения SQL в качестве сопоставленных атрибутов, например, ниже, где мы создаем атрибут **fullname**, которое представляет собой конкатенацию строк столбцов **firstname** и **lastname**:

```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    firstname = Column(String(50))
    lastname = Column(String(50))
    fullname = column_property(firstname + " " + lastname)
```

См. примеры такого использования в [SQL-выражениях как сопоставленных атрибутах](sql-vyrazheniya-kak-mapped-atributy.md).

| Имя объекта                                                                                                                            | Описание                                                            |
| -------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| [column\_property](https://docs.sqlalchemy.org/en/14/orm/mapping\_columns.html#sqlalchemy.orm.column\_property)(\*columns, \*\*kwargs) | Укажите свойство уровня столбца для использования с сопоставлением. |

### _function_ sqlalchemy.orm.column\_property(_\*columns_, _\*\*kwargs_)

Укажите свойство уровня столбца для использования с сопоставлением.

Свойства на основе столбцов обычно можно применять к словарю mapper `properties`, используя элемент [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) напрямую. Используйте эту функцию, когда данный столбец не присутствует непосредственно в выбираемых mapper; примеры включают выражения SQL, функции и скалярные запросы **SELECT**.

Функция **column\_property()** возвращает экземпляр [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty).

Столбцы, которых нет в выбираемых mapper, не будут сохраняться mapper и фактически являются атрибутами «только для чтения».

#### Параметры

* **\*cols (columns)** - список объектов Column для сопоставления.
* **active\_history=False** - Значение `True` указывает, что «предыдущее» значение для скалярного атрибута должно быть загружено при замене, если оно еще не загружено. Обычно логика отслеживания истории для простых скалярных значений, не являющихся первичными ключами, должна знать только «новое» значение, чтобы выполнить сброс. Этот флаг доступен для приложений, которые используют [get\_history()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.attributes.get\_history) или [Session.is\_modified()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.is\_modified), которым также необходимо знать «предыдущее» значение атрибута.
* **comparator\_factory** - класс, который расширяет [Comparator](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty.Comparator), который обеспечивает генерацию пользовательского предложения SQL для операций сравнения.
* **group** - имя группы для этого свойства, когда оно помечено как отложенное.
* **deferred** - когда `True`, свойство столбца является «отложенным», что означает, что оно не загружается сразу, а вместо этого загружается при первом доступе к атрибуту в экземпляре. См. также [deferred()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.deferred).
* **doc** - необязательная строка, которая будет применяться в качестве документа для дескриптора, привязанного к классу.
* **expire\_on\_flush=True** - Отключить истечение срока при сбросе. Свойство **column\_property()**, которое ссылается на выражение SQL (а не на один столбец, привязанный к таблице), считается свойством «только для чтения»; его заполнение не влияет на состояние данных и может возвращать только состояние базы данных. По этой причине срок действия свойства **column\_property()** истекает каждый раз, когда родительский объект участвует в сбросе, то есть имеет какое-либо «грязное» состояние в рамках сброса. Установка для этого параметра значения `False` приведет к тому, что любое существующее значение останется существующим после продолжения сброса. Однако обратите внимание, что сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) с настройками истечения срока действия по умолчанию по-прежнему истекает для всех атрибутов после вызова [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit).
* **info** - Необязательный словарь данных, который будет заполнен атрибутом [MapperProperty.info](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty.info) этого объекта.
* **raiseload** - если `True`, указывает, что столбец должен вызывать ошибку при отмене задержки, а не при загрузке значения. Это можно изменить во время запроса, используя опцию [deferred()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.deferred) с `raiseload=False`.

{% hint style="warning" %}
**Новое в версии 1.4.**
{% endhint %}

{% hint style="info" %}
Смотри также:

[Raiseload for Deferred Columns](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#deferred-raiseload)
{% endhint %}

{% hint style="info" %}
Смотри также:

[Using column\_property for column level options](https://docs.sqlalchemy.org/en/14/orm/mapping\_columns.html#column-property-options) - для сопоставления столбцов с включением параметров сопоставления

[Using column\_property](https://docs.sqlalchemy.org/en/14/orm/mapped\_sql\_expr.html#mapper-column-property-sql-expressions) - для сопоставления выражений SQL
{% endhint %}

## Отображение подмножества столбцов таблицы

Иногда объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) был доступен с использованием процесса отражения, описанного в разделе «**Отражение объектов базы данных**», для загрузки структуры таблицы из базы данных. Для такой таблицы с большим количеством столбцов, на которые не нужно ссылаться в приложении, аргументы **include\_properties** или **exclude\_properties** могут указывать, что должно отображаться только подмножество столбцов. Например:

```python
class User(Base):
    __table__ = user_table
    __mapper_args__ = {
        'include_properties' :['user_id', 'user_name']
    }
```

…сопоставит класс **User** с таблицей **user\_table**, включая только столбцы **user\_id** и **user\_name** — на остальные не ссылаются. Так же:

```python
class Address(Base):
    __table__ = address_table
    __mapper_args__ = {
        'exclude_properties' : ['street', 'city', 'state', 'zip']
    }
```

…сопоставит класс **Address** с таблицей **address\_table**, включая все присутствующие столбцы, кроме **street**, **city**, **state** и **zip**.

Когда используется это сопоставление, столбцы, которые не включены, не будут упоминаться ни в каких операторах **SELECT**, выдаваемых запросом [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query), и не будет никакого сопоставленного атрибута в сопоставленном классе, который представляет столбец; назначение атрибута с таким именем не будет иметь никакого эффекта, кроме обычного назначения атрибута Python.

В некоторых случаях несколько столбцов могут иметь одно и то же имя, например, при сопоставлении с объединением двух или более таблиц, имеющих общее имя столбца. **include\_properties** и **exclude\_properties** также могут содержать объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) для более точного описания того, какие столбцы должны быть включены или исключены:

```python
class UserAddress(Base):
    __table__ = user_table.join(addresses_table)
    __mapper_args__ = {
        'exclude_properties' :[address_table.c.id],
        'primary_key' : [user_table.c.id]
    }
```

{% hint style="info" %}
значения по умолчанию для вставки и обновления, настроенные для отдельных объектов столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), т. е. те, которые описаны в **столбцах INSERT/UPDATE по умолчанию**, включая параметры, настроенные параметрами [Column.default](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.default), [Column.onupdate](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.onupdate), [Column.server\_default](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.server\_default) и [Column.server\_onupdate](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.server\_onupdate), будут продолжать нормально функционировать, даже если эти столбцы объекты не отображаются. Это связано с тем, что в случае [Column.default](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.default) и [Column.onupdate](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.onupdate) объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) все еще присутствует в базовой таблице, что позволяет выполнять функции по умолчанию, когда ORM выдает **INSERT** или **UPDATE**, а в случае [Column.server\_default](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.server\_default) и [Column.server\_onupdate](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.server\_onupdate), сама реляционная база данных выдает эти значения по умолчанию как поведение на стороне сервера.
{% endhint %}
