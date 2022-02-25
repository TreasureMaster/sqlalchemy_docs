# Вставка строк с Core

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Эта страница является частью [учебника по SQLAlchemy 1.4/2.0](../).

Предыдущий: [Работа с данными](./) | Далее: [Выбор строк с помощью Core или ORM](vybor-strok-s-pomoshyu-core-ili-orm.md)
{% endhint %}

При использовании Core оператор SQL **INSERT** генерируется с помощью функции [insert()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.insert) — эта функция создает новый экземпляр [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert), который представляет оператор **INSERT** в SQL, добавляющий новые данные в таблицу.

{% hint style="success" %}
**Для читателей ORM**. Способ, которым строки вставляются **INSERT** в базу данных с точки зрения ORM, использует объектно-ориентированные API-интерфейсы для объекта [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), известного как единица рабочего процесса ([unit of work](https://docs.sqlalchemy.org/en/14/glossary.html#term-unit-of-work)), и существенно отличается от описанного здесь подхода только для Core. Разделы, в большей степени ориентированные на ORM, позже начинаются с «[Вставка строк с помощью ORM](../manipulyacii-s-dannymi-s-pomoshyu-orm.md#vstavka-strok-s-orm)» после разделов «Язык выражений».
{% endhint %}

## Конструкция SQL-выражения insert()

Простой пример [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert), иллюстрирующий целевую таблицу и предложение **VALUES** одновременно:

```python
>>> from sqlalchemy import insert
>>> stmt = insert(user_table).values(name='spongebob', fullname="Spongebob Squarepants")
```

Приведенная выше переменная **stmt** является экземпляром [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert). Большинство выражений SQL можно преобразовать в строки, чтобы увидеть общую форму того, что создается:

```python
>>> print(stmt)
```

```sql
INSERT INTO user_account (name, fullname) VALUES (:name, :fullname)
```

Строковая форма создается путем создания скомпилированной [Compiled](https://docs.sqlalchemy.org/en/14/core/internals.html#sqlalchemy.engine.Compiled) формы объекта, которая включает строковое SQL-представление оператора для конкретной базы данных; мы можем получить этот объект напрямую, используя метод [ClauseElement.compile()](https://docs.sqlalchemy.org/en/14/core/foundation.html#sqlalchemy.sql.expression.ClauseElement.compile):

```python
>>> compiled = stmt.compile()
```

Наша конструкция [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert) является примером «параметризованной» конструкции, проиллюстрированной ранее в разделе «[Отправка параметров](../rabota-s-tranzakciyami-i-dbapi.md#otpravka-parametrov)»; чтобы просмотреть [связанные параметры](https://docs.sqlalchemy.org/en/14/glossary.html#term-bound-parameters) имени **name** и полного имени **fullname**, они также доступны из конструкции [Compiled](https://docs.sqlalchemy.org/en/14/core/internals.html#sqlalchemy.engine.Compiled):

```python
>>> compiled.params
{'name': 'spongebob', 'fullname': 'Spongebob Squarepants'}
```

## Выполнение инструкции statement

Вызывая оператор **stmt**, мы можем вставить **INSERT** строку в **user\_table**. **INSERT** SQL, а также связанные параметры можно увидеть в журнале SQL:

```python
>>> with engine.connect() as conn:
...     result = conn.execute(stmt)
...     conn.commit()
```

```sql
BEGIN (implicit)
INSERT INTO user_account (name, fullname) VALUES (?, ?)
[...] ('spongebob', 'Spongebob Squarepants')
COMMIT
```

В приведенной выше простой форме оператор **INSERT** не возвращает никаких строк, и если вставляется только одна строка, он обычно включает возможность возврата информации о значениях по умолчанию на уровне столбца, которые были сгенерированы во время **INSERT** этой строки, в большинстве случаев; обычно целочисленное значение первичного ключа. В приведенном выше случае первая строка в базе данных SQLite обычно возвращает **1** для первого целочисленного значения первичного ключа, которое мы можем получить с помощью метода доступа [CursorResult.inserted\_primary\_key](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.inserted\_primary\_key):

```python
>>> result.inserted_primary_key
(1,)
```

{% hint style="info" %}
Совет:

[CursorResult.inserted\_primary\_key](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.inserted\_primary\_key) возвращает кортеж, поскольку первичный ключ может содержать несколько столбцов. Это известно как [составной первичный ключ](https://docs.sqlalchemy.org/en/14/glossary.html#term-composite-primary-key). [CursorResult.inserted\_primary\_key](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.inserted\_primary\_key) предназначен для того, чтобы всегда содержать полный первичный ключ только что вставленной записи, а не только значение типа «**cursor.lastrowid**», а также предназначен для заполнения независимо от того, использовался ли автоинкремент (**autoincrement**), следовательно, чтобы выразить полный первичный ключ, для этого нужен кортеж.
{% endhint %}

{% hint style="danger" %}
**Изменено в версии 1.4.8**: кортеж, возвращаемый [CursorResult.inserted\_primary\_key](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.inserted\_primary\_key), теперь является **именованным кортежем**, который выполняется путем возврата его как объекта [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row).
{% endhint %}

## INSERT обычно автоматически генерирует предложение «values»

В приведенном выше примере использовался метод [Insert.values()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert.values) для явного создания предложения **VALUES** инструкции SQL **INSERT**. На самом деле у этого метода есть несколько вариантов, которые допускают специальные формы, такие как несколько строк в одном выражении и вставка выражений SQL. Однако обычный способ использования [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert) заключается в том, что предложение **VALUES** генерируется автоматически из параметров, переданных методу [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute); ниже мы вставляем **INSERT** еще две строки, чтобы проиллюстрировать это:

```python
>>> with engine.connect() as conn:
...     result = conn.execute(
...         insert(user_table),
...         [
...             {"name": "sandy", "fullname": "Sandy Cheeks"},
...             {"name": "patrick", "fullname": "Patrick Star"}
...         ]
...     )
...     conn.commit()
```

```sql
BEGIN (implicit)
INSERT INTO user_account (name, fullname) VALUES (?, ?)
[...] (('sandy', 'Sandy Cheeks'), ('patrick', 'Patrick Star'))
COMMIT
```

Приведенное выше выполнение имеет форму «**executemany**», впервые проиллюстрированную в разделе «[Отправка нескольких параметров](../rabota-s-tranzakciyami-i-dbapi.md#otpravka-neskolkikh-parametrov)», однако, в отличие от использования конструкции [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text), нам не нужно было указывать какой-либо SQL. Передавая словарь или список словарей в метод [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute) в сочетании с конструкцией [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert), [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection) гарантирует, что передаваемые имена столбцов будут автоматически выражены в предложении **VALUES** конструкции [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert).

{% hint style="info" %}
**Глубокая Алхимия**

Привет, добро пожаловать в первый выпуск **Deep Alchemy**. Человек слева известен как **Алхимик**, и вы заметите, что он **не** волшебник, так как остроконечная шляпа не торчит вверх. Алхимик приходит, чтобы описать вещи, которые обычно являются **более продвинутыми и/или сложными** и, кроме того, **обычно не нужными**, но по какой-то причине, по их мнению, вы должны знать об этой вещи, которую может сделать SQLAlchemy.

В этом выпуске, чтобы иметь некоторые интересные данные в **address\_table**, ниже приведен более сложный пример, иллюстрирующий, как метод [Insert.values()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert.values) может использоваться явно, в то же время включая дополнительные **VALUES**, сгенерированные из параметров. Скалярный подзапрос ([scalar subquery](https://docs.sqlalchemy.org/en/14/glossary.html#term-scalar-subquery)) строится с использованием конструкции [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), представленной в следующем разделе, и параметры, используемые в подзапросе, настраиваются с использованием явного связанного имени параметра, установленного с помощью конструкции [bindparam()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.bindparam).

Это немного **более глубокая** алхимия, чтобы мы могли добавлять связанные строки, не загружая идентификаторы первичных ключей из операции **user\_table** в приложение. Большинство алхимиков просто используют ORM, который позаботится о таких вещах за нас.
{% endhint %}

```python
>>> from sqlalchemy import select, bindparam
>>> scalar_subq = (
...     select(user_table.c.id).
...     where(user_table.c.name==bindparam('username')).
...     scalar_subquery()
... )

>>> with engine.connect() as conn:
...     result = conn.execute(
...         insert(address_table).values(user_id=scalar_subq),
...         [
...             {"username": 'spongebob', "email_address": "spongebob@sqlalchemy.org"},
...             {"username": 'sandy', "email_address": "sandy@sqlalchemy.org"},
...             {"username": 'sandy', "email_address": "sandy@squirrelpower.org"},
...         ]
...     )
...     conn.commit()
```

```sql
BEGIN (implicit)
INSERT INTO address (user_id, email_address) VALUES ((SELECT user_account.id
FROM user_account
WHERE user_account.name = ?), ?)
[...] (('spongebob', 'spongebob@sqlalchemy.org'), ('sandy', 'sandy@sqlalchemy.org'),
('sandy', 'sandy@squirrelpower.org'))
COMMIT
```

## INSERT…FROM SELECT

Конструкция [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert) может составлять **INSERT**, который получает строки непосредственно из **SELECT** с помощью метода [Insert.from\_select()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert.from\_select):

```python
>>> select_stmt = select(user_table.c.id, user_table.c.name + "@aol.com")
>>> insert_stmt = insert(address_table).from_select(
...     ["user_id", "email_address"], select_stmt
... )
>>> print(insert_stmt)
```

```sql
INSERT INTO address (user_id, email_address)
SELECT user_account.id, user_account.name || :name_1 AS anon_1
FROM user_account
```

## INSERT…RETURNING

Предложение **RETURNING** для поддерживаемых бэкендов используется автоматически для получения последнего вставленного значения первичного ключа, а также значений по умолчанию для сервера. Однако предложение **RETURNING** также может быть указано явно с помощью метода [Insert.returning()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert.returning); в этом случае объект [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result), возвращаемый при выполнении оператора, имеет строки, которые можно получить:

```python
>>> insert_stmt = insert(address_table).returning(address_table.c.id, address_table.c.email_address)
>>> print(insert_stmt)
```

```sql
INSERT INTO address (id, user_id, email_address)
VALUES (:id, :user_id, :email_address)
RETURNING address.id, address.email_address
```

Его также можно комбинировать с [Insert.from\_select()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert.from\_select), как в приведенном ниже примере, основанном на примере, указанном в [**INSERT…FROM SELECT**](vstavka-strok-s-core.md#insert...from-select):

```python
>>> select_stmt = select(user_table.c.id, user_table.c.name + "@aol.com")
>>> insert_stmt = insert(address_table).from_select(
...     ["user_id", "email_address"], select_stmt
... )
>>> print(insert_stmt.returning(address_table.c.id, address_table.c.email_address))
```

```sql
INSERT INTO address (user_id, email_address)
SELECT user_account.id, user_account.name || :name_1 AS anon_1
FROM user_account RETURNING address.id, address.email_address
```

{% hint style="info" %}
**Совет**:

Функция **RETURNING** также поддерживается операторами **UPDATE** и **DELETE**, которые будут представлены позже в этом руководстве. Функция **RETURNING** обычно **\[ 1 ]** поддерживается только для выполнения инструкций, использующих один набор связанных параметров; то есть он не будет работать с формой «**executemany**», представленной в разделе «[Отправка нескольких параметров](../rabota-s-tranzakciyami-i-dbapi.md#otpravka-neskolkikh-parametrov)». Кроме того, некоторые диалекты, такие как диалект **Oracle**, позволяют **RETURNING** возвращать только одну строку в целом, что означает, что он не будет работать с «**INSERT..FROM SELECT**» и не будет работать с многострочными формами обновления [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update) или удаления [Delete](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete).

**\[ 1 ]**

Существует внутренняя поддержка диалекта [psycopg2](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html#module-sqlalchemy.dialects.postgresql.psycopg2) для вставки **INSERT** многих строк одновременно, а также поддержка возврата **RETURNING**, которая используется SQLAlchemy ORM. Однако эта функция не была распространена на все диалекты и еще не является частью обычного API SQLAlchemy.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert) — в документации по SQL Expression API
{% endhint %}

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел руководства: [Выбор строк с помощью Core или ORM](vybor-strok-s-pomoshyu-core-ili-orm.md)
{% endhint %}
