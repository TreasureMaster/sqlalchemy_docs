# Руководство по запросам ORM

В этом разделе представлен обзор создания запросов с помощью SQLAlchemy ORM с использованием [стиля 2.0](https://docs.sqlalchemy.org/en/14/glossary.html#term-2.0-style).

Читатели этого раздела должны быть знакомы с обзором SQLAlchemy в [SQLAlchemy 1.4 / 2.0 Tutorial](../../sqlalchemy-2.0/uchebnik-po-sqlalchemy-1.4-2.0/), и, в частности, большая часть содержимого здесь расширяет содержимое раздела [Выбор строк с помощью Core или ORM](../../sqlalchemy-2.0/uchebnik-po-sqlalchemy-1.4-2.0/rabota-s-dannymi/vybor-strok-s-pomoshyu-core-ili-orm.md).

## ОПЕРАТОРЫ SELECT

Операторы **SELECT** создаются функцией [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), которая возвращает объект [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select):

```python
>>> from sqlalchemy import select
>>> stmt = select(User).where(User.name == 'spongebob')
```

Чтобы вызвать [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select) с помощью ORM, он передается в [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute):

```python
>>> result = session.execute(stmt)
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = ?
[...] ('spongebob',)
```

```python
>>> for user_obj in result.scalars():
...     print(f"{user_obj.name} {user_obj.fullname}")
spongebob Spongebob Squarepants
```

## Выбор объектов и атрибутов ORM

Конструкция [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select) принимает **объекты ORM**, включая **сопоставленные классы**, а также атрибуты уровня класса, представляющие **сопоставленные столбцы**, которые во время создания преобразуются в ORM-аннотированные элементы [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause) и [ColumnElement](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnElement).

Объект [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), который содержит объекты с аннотациями ORM, обычно выполняется с использованием объекта [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), а не объекта [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection), поэтому могут действовать функции, связанные с ORM, в том числе могут быть возвращены экземпляры объектов, сопоставленных с ORM. При непосредственном использовании [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection) строки результатов будут содержать только данные на уровне столбцов.

Ниже мы выбираем из сущности **User**, создавая [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), который выбирает из сопоставленной таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), с которой сопоставлен пользователь **User**:

```python
>>> result = session.execute(select(User).order_by(User.id))
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account ORDER BY user_account.id
[...] ()
```

При выборе из сущностей ORM сама **сущность возвращается** в результате **в виде строки с одним элементом, а не в виде набора отдельных столбцов**; например выше, [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result) возвращает объекты [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row), которые имеют только один элемент в строке, этот элемент содержит объект **User**:

```python
>>> result.fetchone()
(User(id=1, name='spongebob', fullname='Spongebob Squarepants'),)
```

При выборе списка одноэлементных строк, содержащих сущности ORM, обычно **пропускается генерация** объектов [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row) и вместо этого **получаются сущности ORM напрямую**, что достигается с помощью метода [Result.scalars()](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result.scalars):

```python
>>> result.scalars().all()
[User(id=2, name='sandy', fullname='Sandy Cheeks'),
 User(id=3, name='patrick', fullname='Patrick Star'),
 User(id=4, name='squidward', fullname='Squidward Tentacles'),
 User(id=5, name='ehkrabs', fullname='Eugene H. Krabs')]
```

Сущности ORM **именуются** в строке результатов **на основе имени их класса**, например, ниже, где мы **SELECT** одновременно и пользователя **User**, и адрес **Address**:

```python
>>> stmt = select(User, Address).join(User.addresses).order_by(User.id, Address.id)

>>> for row in session.execute(stmt):
...    print(f"{row.User.name} {row.Address.email_address}")
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname,
address.id AS id_1, address.user_id, address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
ORDER BY user_account.id, address.id
[...] ()
```

```bash
spongebob spongebob@sqlalchemy.org
sandy sandy@sqlalchemy.org
sandy squirrel@squirrelpower.org
patrick pat999@aol.com
squidward stentcl@sqlalchemy.org
```

### Выбор отдельных атрибутов

Атрибуты сопоставленного класса, такие как **User.name** и **Address.email\_address**, ведут себя так же, как и поведение самого класса сущностей, такого как **User**, в том смысле, что они _**автоматически преобразуются**_ в ORM-аннотированные объекты **Core** при передаче в [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select). Их можно использовать так же, как столбцы таблицы:

```python
>>> result = session.execute(
...     select(User.name, Address.email_address).
...     join(User.addresses).
...     order_by(User.id, Address.id)
... )
```

```sql
SELECT user_account.name, address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
ORDER BY user_account.id, address.id
[...] ()
```

Атрибуты ORM, известные как объекты [InstrumentedAttribute](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.InstrumentedAttribute), могут использоваться так же, как и любой [ColumnElement](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnElement), и точно так же доставляются в строках результатов, например, ниже, где мы ссылаемся на их значения по имени столбца в каждой строке:

```python
>>> for row in result:
...     print(f"{row.name}  {row.email_address}")
spongebob  spongebob@sqlalchemy.org
sandy  sandy@sqlalchemy.org
sandy  squirrel@squirrelpower.org
patrick  pat999@aol.com
squidward  stentcl@sqlalchemy.org
```

### Группировка выбранных атрибутов с помощью пакетов

Конструкция [Bundle](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Bundle) — это расширяемая конструкция, предназначенная _**только для ORM**_, которая **позволяет группировать наборы выражений столбцов** в строках результатов:

```python
>>> from sqlalchemy.orm import Bundle
>>> stmt = select(
...     Bundle("user", User.name, User.fullname),
...     Bundle("email", Address.email_address)
... ).join_from(User, Address)

>>> for row in session.execute(stmt):
...     print(f"{row.user.name} {row.email.email_address}")
```

```sql
SELECT user_account.name, user_account.fullname, address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
[...] ()
```

```bash
spongebob spongebob@sqlalchemy.org
sandy sandy@sqlalchemy.org
sandy squirrel@squirrelpower.org
patrick pat999@aol.com
squidward stentcl@sqlalchemy.org
```

[Bundle](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Bundle) потенциально полезен для создания упрощенных представлений, а также пользовательских групп столбцов, таких как сопоставления.

{% hint style="info" %}
Смотри также:

[Пакеты столбцов](zagruzka-stolbcov.md#pakety-stolbcov) — в документации по загрузке ORM.
{% endhint %}

### Выбор псевдонимов ORM

Как обсуждалось в руководстве по [использованию псевдонимов](../../sqlalchemy-2.0/uchebnik-po-sqlalchemy-1.4-2.0/rabota-s-dannymi/vybor-strok-s-pomoshyu-core-ili-orm.md#ispolzovanie-psevdonimov-alias), создание псевдонима SQL объекта ORM достигается с помощью конструкции [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) для сопоставленного класса:

```python
>>> from sqlalchemy.orm import aliased
>>> u1 = aliased(User)
>>> print(select(u1).order_by(u1.id))
```

```sql
SELECT user_account_1.id, user_account_1.name, user_account_1.fullname
FROM user_account AS user_account_1 ORDER BY user_account_1.id
```

Как и в случае использования [Table.alias()](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table.alias), псевдониму SQL присваивается анонимное имя. В случае выбора объекта из строки с явным именем можно также передать параметр [aliased.name](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased.params.name):

```python
>>> from sqlalchemy.orm import aliased
>>> u1 = aliased(User, name="u1")
>>> stmt = select(u1).order_by(u1.id)

>>> row = session.execute(stmt).first()
```

```sql
SELECT u1.id, u1.name, u1.fullname
FROM user_account AS u1 ORDER BY u1.id
[...] ()
```

```python
>>> print(f"{row.u1.name}")
spongebob
```

Конструкция с псевдонимом [aliased](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) также играет центральную роль в использовании подзапросов с ORM; в разделах «[Выбор объектов из подзапросов](rukovodstvo-po-zaprosam-orm.md#vybor-sushnostei-iz-podzaprosov)» и «[Присоединение к подзапросам](rukovodstvo-po-zaprosam-orm.md#prisoedinenie-k-podzaprosam)» это обсуждается подробнее.

### Получение результатов ORM из текстовых и выражений Core

ORM поддерживает загрузку сущностей из операторов **SELECT**, поступающих из других источников. Типичным вариантом использования является текстовый оператор **SELECT**, который в SQLAlchemy представлен с помощью конструкции [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text). Конструкция [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text) после создания может быть дополнена информацией о столбцах, отображаемых с помощью ORM, которые будут загружены оператором; затем это можно связать с самим объектом ORM, чтобы объекты ORM можно было загружать на основе этого оператора.

Учитывая текстовый оператор SQL, который мы хотели бы загрузить из:

```python
>>> from sqlalchemy import text
>>> textual_sql = text("SELECT id, name, fullname FROM user_account ORDER BY id")
```

Мы можем добавить информацию о столбцах в инструкцию, используя метод [TextClause.columns()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.TextClause.columns); при вызове этого метода объект [TextClause](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.TextClause) преобразуется в объект [TextualSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.TextualSelect), который берет на себя роль, сравнимую с конструкцией [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select). Методу [TextClause.columns()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.TextClause.columns) обычно передаются объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) или их эквиваленты, и в этом случае мы можем напрямую использовать атрибуты ORM-отображения в классе **User**:

```python
>>> textual_sql = textual_sql.columns(User.id, User.name, User.fullname)
```

Теперь у нас есть конструкция SQL, сконфигурированная с помощью ORM, которая, как указано, может загружать столбцы «**id**», «**name**» и «**fullname**» отдельно. Чтобы вместо этого использовать этот оператор **SELECT** в качестве источника полных сущностей **User**, мы можем связать эти столбцы с обычной конструкцией [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select) с поддержкой ORM, используя метод [Select.from\_statement()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.from\_statement):

```python
>>> # использование from_statement()
>>> orm_sql = select(User).from_statement(textual_sql)
>>> for user_obj in session.execute(orm_sql).scalars():
...     print(user_obj)
```

```sql
SELECT id, name, fullname FROM user_account ORDER BY id
[...] ()
```

```python
User(id=1, name='spongebob', fullname='Spongebob Squarepants')
User(id=2, name='sandy', fullname='Sandy Cheeks')
User(id=3, name='patrick', fullname='Patrick Star')
User(id=4, name='squidward', fullname='Squidward Tentacles')
User(id=5, name='ehkrabs', fullname='Eugene H. Krabs')
```

Тот же объект [TextualSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.TextualSelect) также можно преобразовать в подзапрос с помощью метода [TextualSelect.subquery()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.TextualSelect.subquery) и связать с ним сущность **User** с помощью конструкции [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) аналогично описанному ниже в разделе [Выбор сущностей из подзапросов](rukovodstvo-po-zaprosam-orm.md#vybor-sushnostei-iz-podzaprosov):

```python
>>> # использование aliased() для выборки из подзапроса
>>> orm_subquery = aliased(User, textual_sql.subquery())
>>> stmt = select(orm_subquery)
>>> for user_obj in session.execute(stmt).scalars():
...     print(user_obj)
```

```sql
SELECT anon_1.id, anon_1.name, anon_1.fullname
FROM (SELECT id, name, fullname FROM user_account ORDER BY id) AS anon_1
[...] ()
```

```python
User(id=1, name='spongebob', fullname='Spongebob Squarepants')
User(id=2, name='sandy', fullname='Sandy Cheeks')
User(id=3, name='patrick', fullname='Patrick Star')
User(id=4, name='squidward', fullname='Squidward Tentacles')
User(id=5, name='ehkrabs', fullname='Eugene H. Krabs')
```

Разница между использованием [TextualSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.TextualSelect) непосредственно с [Select.from\_statement()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.from\_statement) и использованием **aliased()** заключается в том, что в первом случае в результирующем SQL не создается подзапрос. В некоторых сценариях это может быть выгодно с точки зрения производительности или сложности.

{% hint style="info" %}
Смотри также:

[Using INSERT, UPDATE and ON CONFLICT (i.e. upsert) to return ORM Objects](https://docs.sqlalchemy.org/en/14/orm/persistence\_techniques.html#orm-dml-returning-objects) - Метод [Select.from\_statement()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.from\_statement) также работает с операторами [DML](https://docs.sqlalchemy.org/en/14/glossary.html#term-DML), поддерживающими **RETURNING**.
{% endhint %}

## Выбор сущностей из подзапросов

Конструкцию [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased), обсуждавшуюся в предыдущем разделе, можно использовать с любой конструкцией **Subquery**, полученной из такого метода, как [Select.subquery()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.subquery), чтобы связать объекты ORM со столбцами, возвращаемыми этим подзапросом; **должно существовать** отношение соответствия столбцов между столбцами, предоставленными подзапросом, и столбцами, на которые сопоставляется сущность, что означает, что подзапрос должен быть в конечном счете получен из этих сущностей, как в приведенном ниже примере:

```python
>>> inner_stmt = select(User).where(User.id < 7).order_by(User.id)
>>> subq = inner_stmt.subquery()
>>> aliased_user = aliased(User, subq)
>>> stmt = select(aliased_user)
>>> for user_obj in session.execute(stmt).scalars():
...     print(user_obj)
```

```sql
SELECT anon_1.id, anon_1.name, anon_1.fullname
FROM (SELECT user_account.id AS id, user_account.name AS name, user_account.fullname AS fullname
FROM user_account
WHERE user_account.id < ? ORDER BY user_account.id) AS anon_1
[generated in ...] (7,)
```

```python
User(id=1, name='spongebob', fullname='Spongebob Squarepants')
User(id=2, name='sandy', fullname='Sandy Cheeks')
User(id=3, name='patrick', fullname='Patrick Star')
User(id=4, name='squidward', fullname='Squidward Tentacles')
User(id=5, name='ehkrabs', fullname='Eugene H. Krabs')
```

{% hint style="info" %}
Смотри также:

[Подзапросы/CTE сущностей ORM](../../sqlalchemy-2.0/uchebnik-po-sqlalchemy-1.4-2.0/rabota-s-dannymi/vybor-strok-s-pomoshyu-core-ili-orm.md#podzaprosy-cte-sushnostei-orm) - в документации [SQLAlchemy 1.4 / 2.0 Tutorial](https://docs.sqlalchemy.org/en/14/tutorial/index.html#unified-tutorial)

[Joining to Subqueries](https://docs.sqlalchemy.org/en/14/orm/queryguide.html#orm-queryguide-join-subqueries)
{% endhint %}

## Выбор сущностей из UNION и других операций набора

## Соединения JOIN

### Простые соединения отношений

### Цепочка нескольких объединений

### Присоединение к целевому объекту или выборке selectable

### Присоединение к цели с предложением ON

### Дополнение встроенных предложений ON

### Присоединение к подзапросам

### Контроль того, к чему присоединяться (JOIN FROM)
