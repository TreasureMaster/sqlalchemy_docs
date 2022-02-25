# Обновление и удаление строк с помощью Core

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Эта страница является частью [учебника по SQLAlchemy 1.4/2.0](../).

Предыдущий: [Выбор строк (работа с SQL функциями)](vybor-strok-rabota-s-sql-funkciyami.md) | Далее: [Манипуляции с данными с помощью ORM](../manipulyacii-s-dannymi-s-pomoshyu-orm.md)
{% endhint %}

До сих пор мы рассмотрели [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert), чтобы мы могли получить некоторые данные в нашу базу данных, а затем потратили много времени на [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), который обрабатывает широкий спектр шаблонов использования, используемых для извлечения данных из базы данных. В этом разделе мы рассмотрим конструкции [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update) и [Delete](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete), которые используются для изменения существующих строк, а также для удаления существующих строк. В этом разделе эти конструкции будут рассмотрены с точки зрения Core.

{% hint style="success" %}
**Для читателей ORM**. Как упоминалось в разделе «[Вставка строк с Core](vstavka-strok-s-core.md)», операции [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update) и [Delete](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete) при использовании с ORM обычно вызываются внутренне из объекта [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) как часть рабочего процесса ([unit of work](https://docs.sqlalchemy.org/en/14/glossary.html#term-unit-of-work)).

Однако, в отличие от [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert), конструкции [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update) и [Delete](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete) также можно использовать непосредственно с ORM, используя шаблон, известный как «обновление и удаление с поддержкой ORM»; по этой причине знакомство с этими конструкциями полезно для использования ORM. Оба стиля использования обсуждаются в разделах [Обновление объектов ORM](../manipulyacii-s-dannymi-s-pomoshyu-orm.md#obnovlenie-orm-obektov) и [Удаление объектов ORM](../manipulyacii-s-dannymi-s-pomoshyu-orm.md#udalenie-orm-obektov).
{% endhint %}

## Конструкция SQL-выражения update()

Функция [update()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.update) генерирует новый экземпляр [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update), который представляет оператор **UPDATE** в SQL, который будет обновлять существующие данные в таблице.

Подобно конструкции [insert()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.insert), существует «_традиционная_» форма [update()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.update), которая выдает **UPDATE** для одной таблицы за раз и не возвращает никаких строк. Однако некоторые бэкенды поддерживают оператор **UPDATE**, который может одновременно изменять несколько таблиц, а оператор **UPDATE** также поддерживает **RETURNING**, так что столбцы, содержащиеся в совпадающих строках, могут быть возвращены в результирующем наборе.

Базовое **UPDATE** выглядит так:

```python
>>> from sqlalchemy import update
>>> stmt = (
...     update(user_table).where(user_table.c.name == 'patrick').
...     values(fullname='Patrick the Star')
... )
>>> print(stmt)
```

```sql
UPDATE user_account SET fullname=:fullname WHERE user_account.name = :name_1
```

Метод [Update.values()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update.values) управляет содержимым элементов **SET** инструкции **UPDATE**. Это тот же метод, который используется в конструкции [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert). Обычно параметры можно передавать, используя имена столбцов в качестве аргументов ключевого слова.

**UPDATE** поддерживает все основные SQL-формы **UPDATE**, включая обновления выражений, где мы можем использовать выражения [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column):

```python
>>> stmt = (
...     update(user_table).
...     values(fullname="Username: " + user_table.c.name)
... )
>>> print(stmt)
```

```sql
UPDATE user_account SET fullname=(:name_1 || user_account.name)
```

Для поддержки **UPDATE** в контексте «_**executemany**_», когда для одного и того же оператора будет вызываться множество наборов параметров, для установки связанных параметров может использоваться конструкция [bindparam()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.bindparam); они заменяют места, в которые обычно помещаются литеральные значения:

```python
>>> from sqlalchemy import bindparam
>>> stmt = (
...   update(user_table).
...   where(user_table.c.name == bindparam('oldname')).
...   values(name=bindparam('newname'))
... )
>>> with engine.begin() as conn:
...   conn.execute(
...       stmt,
...       [
...          {'oldname':'jack', 'newname':'ed'},
...          {'oldname':'wendy', 'newname':'mary'},
...          {'oldname':'jim', 'newname':'jake'},
...       ]
...   )
```

```sql
BEGIN (implicit)
UPDATE user_account SET name=? WHERE user_account.name = ?
[...] (('ed', 'jack'), ('mary', 'wendy'), ('jake', 'jim'))
<sqlalchemy.engine.cursor.CursorResult object at 0x...>
COMMIT
```

Другие методы, которые могут быть применены к **UPDATE**, включают:

### Коррелированные обновления

Оператор **UPDATE** может использовать строки в других таблицах с помощью [коррелированного подзапроса](vybor-strok-s-pomoshyu-core-ili-orm.md#skalyarnye-i-korrelirovannye-podzaprosy). Подзапрос можно использовать везде, где может быть размещено выражение столбца:

```python
>>> scalar_subq = (
...   select(address_table.c.email_address).
...   where(address_table.c.user_id == user_table.c.id).
...   order_by(address_table.c.id).
...   limit(1).
...   scalar_subquery()
... )
>>> update_stmt = update(user_table).values(fullname=scalar_subq)
>>> print(update_stmt)
```

```sql
UPDATE user_account SET fullname=(SELECT address.email_address
FROM address
WHERE address.user_id = user_account.id ORDER BY address.id
LIMIT :param_1)
```

### UPDATE..FROM

Некоторые базы данных, такие как **PostgreSQL** и **MySQL**, поддерживают синтаксис «**UPDATE FROM**», в котором дополнительные таблицы могут быть указаны непосредственно в специальном предложении **FROM**. Этот синтаксис будет сгенерирован неявно, когда дополнительные таблицы расположены в предложении **WHERE** оператора:

```python
>>> update_stmt = (
...    update(user_table).
...    where(user_table.c.id == address_table.c.user_id).
...    where(address_table.c.email_address == 'patrick@aol.com').
...    values(fullname='Pat')
...  )
>>> print(update_stmt)
```

```sql
UPDATE user_account SET fullname=:fullname FROM address
WHERE user_account.id = address.user_id AND address.email_address = :email_address_1
```

Существует также специальный синтаксис **MySQL**, который может **UPDATE** несколько таблиц. Это требует, чтобы мы ссылались на объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) в предложении **VALUES**, чтобы ссылаться на дополнительные таблицы:

```python
>>> update_stmt = (
...    update(user_table).
...    where(user_table.c.id == address_table.c.user_id).
...    where(address_table.c.email_address == 'patrick@aol.com').
...    values(
...        {
...            user_table.c.fullname: "Pat",
...            address_table.c.email_address: "pat@aol.com"
...        }
...    )
...  )
>>> from sqlalchemy.dialects import mysql
>>> print(update_stmt.compile(dialect=mysql.dialect()))
```

```sql
UPDATE user_account, address
SET address.email_address=%s, user_account.fullname=%s
WHERE user_account.id = address.user_id AND address.email_address = %s
```

### Упорядоченные обновления параметров

Еще одно поведение, характерное только для **MySQL**, заключается в том, что порядок параметров в предложении **SET** оператора **UPDATE** фактически влияет на оценку каждого выражения. В этом случае метод [Update.ordered\_values()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update.ordered\_values) принимает последовательность кортежей, чтобы можно было управлять этим порядком **\[ 2 ]**:

```python
>>> update_stmt = (
...     update(some_table).
...     ordered_values(
...         (some_table.c.y, 20),
...         (some_table.c.x, some_table.c.y + 10)
...     )
... )
>>> print(update_stmt)
```

```sql
UPDATE some_table SET y=:y, x=(some_table.y + :y_1)
```

{% hint style="info" %}
**\[ 2 ]**

Хотя словари Python [гарантированно упорядочены вставками](https://mail.python.org/pipermail/python-dev/2017-December/151283.html), начиная с Python 3.7, метод [Update.ordered\_values()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update.ordered\_values) по-прежнему обеспечивает дополнительную меру ясности намерений, когда важно, чтобы предложение **SET** оператора **MySQL** **UPDATE** выполнялось определенным образом.
{% endhint %}

## Конструкция SQL-выражения delete()

Функция [delete()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.delete) создает новый экземпляр [Delete](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete), представляющий оператор **DELETE** в SQL, который удалит строки из таблицы.

Оператор [delete()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.delete) с точки зрения API очень похож на оператор [update()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.update), традиционно не возвращая строк, но допуская вариант **RETURNING** для некоторых серверных частей базы данных.

```python
>>> from sqlalchemy import delete
>>> stmt = delete(user_table).where(user_table.c.name == 'patrick')
>>> print(stmt)
```

```sql
DELETE FROM user_account WHERE user_account.name = :name_1
```

### Несколько удалений из таблиц

Как и [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update), [Delete](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete) поддерживает использование коррелированных подзапросов в предложении **WHERE**, а также специфический для серверной части синтаксис нескольких таблиц, например **DELETE FROM..USING** в **MySQL**:

```python
>>> delete_stmt = (
...    delete(user_table).
...    where(user_table.c.id == address_table.c.user_id).
...    where(address_table.c.email_address == 'patrick@aol.com')
...  )
>>> from sqlalchemy.dialects import mysql
>>> print(delete_stmt.compile(dialect=mysql.dialect()))
```

```sql
DELETE FROM user_account USING user_account, address
WHERE user_account.id = address.user_id AND address.email_address = %s
```

## Получение затронутого количества строк от UPDATE, DELETE

Как [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update), так и [Delete](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete) поддерживают возможность возврата количества строк, совпавших после выполнения оператора, для операторов, которые вызываются с помощью Core [Connection](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Connection), т. е. [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Connection.execute). В соответствии с предостережениями, упомянутыми ниже, это значение доступно из атрибута [CursorResult.rowcount](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.rowcount):

```python
>>> with engine.begin() as conn:
...     result = conn.execute(
...         update(user_table).
...         values(fullname="Patrick McStar").
...         where(user_table.c.name == 'patrick')
...     )
...     print(result.rowcount)
```

```sql
BEGIN (implicit)
UPDATE user_account SET fullname=? WHERE user_account.name = ?
[...] ('Patrick McStar', 'patrick')
```

```python
1
```

```sql
COMMIT
```

{% hint style="info" %}
Совет:

Класс [CursorResult](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult) является подклассом [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result), который содержит дополнительные атрибуты, характерные для объекта курсора **DBAPI**. Экземпляр этого подкласса возвращается, когда оператор вызывается с помощью метода [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Connection.execute). При использовании ORM метод [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute) возвращает объект этого типа для всех инструкций **INSERT**, **UPDATE** и **DELETE**.
{% endhint %}

Факты о [CursorResult.rowcount](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.rowcount):

* Возвращаемое значение — это количество строк, **соответствующих** предложению **WHERE** инструкции. Не имеет значения, действительно ли строка была изменена или нет.
* [CursorResult.rowcount](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.rowcount) _**не обязательно**_ доступен для инструкции **UPDATE** или **DELETE**, использующей **RETURNING**.
* Для выполнения [executemany](../rabota-s-tranzakciyami-i-dbapi.md#otpravka-neskolkikh-parametrov) атрибут [CursorResult.rowcount](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.rowcount) также может быть недоступен, что сильно зависит от используемого модуля **DBAPI**, а также от настроенных параметров. Атрибут [CursorResult.supports\_sane\_multi\_rowcount](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.supports\_sane\_multi\_rowcount) указывает, будет ли это значение доступно для текущего используемого бэкенда.
* Некоторые драйверы, особенно сторонние диалекты для _**нереляционных баз данных**_, могут вообще не поддерживать [CursorResult.rowcount](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.rowcount). [CursorResult.supports\_sane\_rowcount](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.supports\_sane\_rowcount) укажет на это.
* «**rowcount**» используется единицей ([unit of work](https://docs.sqlalchemy.org/en/14/glossary.html#term-unit-of-work)) рабочего процесса ORM для проверки того, что оператор **UPDATE** или **DELETE** соответствует ожидаемому количеству строк, а также необходим для функции управления версиями ORM, описанной в разделе «[Настройка счетчика версий](../../../sqlalchemy-orm/konfiguraciya-mapper/konfigurirovanie-schetchika-versii.md)».

## Использование RETURNING с UPDATE, DELETE

Подобно конструкции [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert), [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update) и [Delete](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete) также поддерживают предложение **RETURNING**, которое добавляется с помощью методов [Update.returning()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update.returning) и [Delete.returning()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete.returning). Когда эти методы используются в серверной части, поддерживающей **RETURNING**, выбранные столбцы из всех строк, которые соответствуют критериям **WHERE** оператора, будут возвращены в объекте [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result) в виде строк, которые можно итерировать:

```python
>>> update_stmt = (
...     update(user_table).where(user_table.c.name == 'patrick').
...     values(fullname='Patrick the Star').
...     returning(user_table.c.id, user_table.c.name)
... )
>>> print(update_stmt)
```

```sql
UPDATE user_account SET fullname=:fullname
WHERE user_account.name = :name_1
RETURNING user_account.id, user_account.name
```

```python
>>> delete_stmt = (
...     delete(user_table).where(user_table.c.name == 'patrick').
...     returning(user_table.c.id, user_table.c.name)
... )
>>> print(delete_stmt)
```

```sql
DELETE FROM user_account
WHERE user_account.name = :name_1
RETURNING user_account.id, user_account.name
```

## Дальнейшее чтение для UPDATE, DELETE

{% hint style="info" %}
Смотри также:

Документация по API для **UPDATE**/**DELETE**:

* [`Update`](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update)
* [`Delete`](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete)

**UPDATE** и **DELETE** с поддержкой ORM:

* [Операторы UPDATE с поддержкой ORM](../manipulyacii-s-dannymi-s-pomoshyu-orm.md#operatory-update-s-podderzhkoi-orm)
* [Операторы DELETE с поддержкой ORM](../manipulyacii-s-dannymi-s-pomoshyu-orm.md#operatory-delete-s-podderzhkoi-orm)
{% endhint %}

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел руководства: [Манипуляции с данными с помощью ORM](../manipulyacii-s-dannymi-s-pomoshyu-orm.md)
{% endhint %}
