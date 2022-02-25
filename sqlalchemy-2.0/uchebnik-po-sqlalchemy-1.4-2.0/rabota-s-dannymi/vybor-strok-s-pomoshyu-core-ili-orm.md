# Выбор строк с помощью Core или ORM

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Эта страница является частью [учебника по SQLAlchemy 1.4/2.0](../).

Предыдущий: [Вставка строк с Core](vstavka-strok-s-core.md) | Далее:  [Выбор строк (работа с SQL функциями)](vybor-strok-rabota-s-sql-funkciyami.md)
{% endhint %}

И для Core, и для ORM функция [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select) генерирует конструкцию [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), которая используется для всех запросов **SELECT**. Передаваемый таким методам, как [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute) в Core и [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute) в ORM, оператор **SELECT** выдается в текущей транзакции, а строки результатов доступны через возвращаемый объект [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result).

{% hint style="success" %}
**Для читателей ORM** — содержимое здесь одинаково хорошо применимо как к использованию Core, так и к использованию ORM, и здесь упоминаются основные варианты использования ORM. Однако доступно гораздо больше функций, специфичных для ORM; они задокументированы в Руководстве по запросам ORM ([ORM Querying Guide](https://docs.sqlalchemy.org/en/14/orm/queryguide.html)).
{% endhint %}

## Конструкция SQL-выражения select()

Конструкция [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select) строит оператор так же, как и [insert()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.insert), используя [генеративный](https://docs.sqlalchemy.org/en/14/glossary.html#term-generative) подход, при котором каждый метод создает больше состояния для объекта. Как и другие конструкции SQL, его можно преобразовать в строку:

```python
>>> from sqlalchemy import select
>>> stmt = select(user_table).where(user_table.c.name == 'spongebob')
>>> print(stmt)
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = :name_1
```

Точно так же, как и во всех других конструкциях SQL на уровне операторов, для фактического запуска оператора мы передаем его методу выполнения. Поскольку оператор **SELECT** возвращает строки, мы всегда можем выполнить итерацию объекта результата, чтобы вернуть объекты [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row):

```python
>>> with engine.connect() as conn:
...     for row in conn.execute(stmt):
...         print(row)
```

```sql
BEGIN (implicit)
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = ?
[...] ('spongebob',)
```

```python
(1, 'spongebob', 'Spongebob Squarepants')
```

```sql
ROLLBACK
```

При использовании ORM, особенно с конструкцией [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), составленной для сущностей ORM, мы захотим выполнить ее с помощью метода [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute) в сеансе [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session); используя этот подход, мы продолжаем получать объекты [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row) из результата, однако эти строки теперь могут включать полные сущности, такие как экземпляры класса **User**, как отдельные элементы в каждой строке:

```python
>>> stmt = select(User).where(User.name == 'spongebob')
>>> with Session(engine) as session:
...     for row in session.execute(stmt):
...         print(row)
```

```sql
BEGIN (implicit)
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = ?
[...] ('spongebob',)
```

```python
(User(id=1, name='spongebob', fullname='Spongebob Squarepants'),)
```

```sql
ROLLBACK
```

{% hint style="info" %}
**select() из таблицы Table против класса ORM**

Хотя SQL, сгенерированный в этих примерах, выглядит одинаково независимо от того, вызываем ли мы **select(user\_table)** или **select(User)**, в более общем случае они не обязательно отображают одно и то же, поскольку класс, отображаемый ORM, может быть сопоставлен с другими типами «**selectables**» помимо таблиц. Вызов **select()**, относящийся к объекту ORM, также указывает, что в результате должны быть возвращены экземпляры, сопоставленные с ORM, чего нельзя сказать о **SELECT** из объекта [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table).
{% endhint %}

В следующих разделах конструкция **SELECT** будет обсуждаться более подробно.

## Установка предложения COLUMNS и FROM

Функция [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select) принимает позиционные элементы, представляющие любое количество выражений [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) и/или [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), а также широкий спектр совместимых объектов, которые преобразуются в список SQL-выражений, из которых следует выбрать **SELECT**, которые будут возвращены как столбцы в набор результатов. Эти элементы также служат в более простых случаях для создания предложения **FROM**, которое выводится из переданных столбцов и табличных выражений:

```python
>>> print(select(user_table))
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
```

Чтобы выбрать **SELECT** из отдельных столбцов с использованием подхода _**Core**_, доступ к объектам столбцов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) осуществляется из средства доступа [Table.c](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table.c), и их можно отправлять напрямую; предложение **FROM** будет рассматриваться как набор всех объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) и других объектов [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause), которые представлены этими столбцами:

```python
>>> print(select(user_table.c.name, user_table.c.fullname))
```

```sql
SELECT user_account.name, user_account.fullname
FROM user_account
```

### Выбор объектов и столбцов ORM

Объекты ORM, такие как наш класс **User**, а также сопоставленные с ним атрибуты столбцов, такие как **User.name**, также участвуют в системе языка выражений SQL, представляющей таблицы и столбцы. Ниже показан пример **SELECT**ing из объекта **User**, который в конечном итоге отображается так же, как если бы мы использовали **user\_table** напрямую:

```python
>>> print(select(User))
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
```

При выполнении оператора, подобного приведенному выше, с использованием метода ORM [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute), существует важное отличие, когда мы выбираем из полного объекта, такого как **User**, в отличие от **user\_table**, который заключается в том, что **сам объект возвращается как один элемент внутри каждой строки**. То есть, когда мы извлекаем строки из приведенного выше оператора, поскольку в списке вещей для выборки есть только сущность **User**, мы возвращаем объекты [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row), которые имеют только один элемент, который содержит экземпляры класса **User**:

```python
>>> row = session.execute(select(User)).first()
```

```sql
BEGIN...
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
[...] ()
```

```python
>>> row
(User(id=1, name='spongebob', fullname='Spongebob Squarepants'),)
```

Вышеупомянутая строка [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row) имеет только один элемент, представляющий сущность пользователя **User**:

```python
>>> row[0]
User(id=1, name='spongebob', fullname='Spongebob Squarepants')
```

В качестве альтернативы мы можем выбрать отдельные столбцы объекта ORM как отдельные элементы в строках результатов, используя атрибуты, привязанные к классу; когда они передаются конструкции, такой как [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), они преобразуются в столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) или другое выражение SQL, представленное каждым атрибутом:

```python
>>> print(select(User.name, User.fullname))
```

```sql
SELECT user_account.name, user_account.fullname
FROM user_account
```

Когда мы вызываем _**этот**_ оператор с помощью [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute), мы теперь получаем строки, которые имеют отдельные элементы для каждого значения, каждый из которых соответствует отдельному столбцу или другому выражению SQL:

```python
>>> row = session.execute(select(User.name, User.fullname)).first()
```

```sql
SELECT user_account.name, user_account.fullname
FROM user_account
[...] ()
```

```python
>>> row
('spongebob', 'Spongebob Squarepants')
```

Подходы также могут быть смешанными, как показано ниже, где мы выбираем **SELECT** атрибут **name** объекта **User** в качестве первого элемента строки и объединяем его с полными объектами **Address** во втором элементе:

```python
>>> session.execute(
...     select(User.name, Address).
...     where(User.id==Address.user_id).
...     order_by(Address.id)
... ).all()
```

```sql
SELECT user_account.name, address.id, address.email_address, address.user_id
FROM user_account, address
WHERE user_account.id = address.user_id ORDER BY address.id
[...] ()
```

```python
[('spongebob', Address(id=1, email_address='spongebob@sqlalchemy.org')),
('sandy', Address(id=2, email_address='sandy@sqlalchemy.org')),
('sandy', Address(id=3, email_address='sandy@squirrelpower.org'))]
```

Подходы к выбору сущностей и столбцов ORM, а также общие методы преобразования строк обсуждаются далее в разделе Выбор сущностей и атрибутов ORM ([Selecting ORM Entities and Attributes](https://docs.sqlalchemy.org/en/14/orm/queryguide.html#orm-queryguide-select-columns)).

{% hint style="info" %}
Смотри также:

[Selecting ORM Entities and Attributes](https://docs.sqlalchemy.org/en/14/orm/queryguide.html#orm-queryguide-select-columns) - в документации [ORM Querying Guide](https://docs.sqlalchemy.org/en/14/orm/queryguide.html)
{% endhint %}

### Выбор из помеченных (labeled) выражений SQL

Метод [ColumnElement.label()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnElement.label), а также одноименный метод, доступный для атрибутов ORM, предоставляет метку SQL для столбца или выражения, позволяя ему иметь определенное имя в результирующем наборе. Это может быть полезно при обращении к произвольным выражениям SQL в строке результата по имени:

```python
>>> from sqlalchemy import func, cast
>>> stmt = (
...     select(
...         ("Username: " + user_table.c.name).label("username"),
...     ).order_by(user_table.c.name)
... )
>>> with engine.connect() as conn:
...     for row in conn.execute(stmt):
...         print(f"{row.username}")
```

```sql
BEGIN (implicit)
SELECT ? || user_account.name AS username
FROM user_account ORDER BY user_account.name
[...] ('Username: ',)
```

```python
Username: patrick
Username: sandy
Username: spongebob
```

```sql
ROLLBACK
```

{% hint style="info" %}
Смотри также:

Упорядочивание или группировка по метке ([Ordering or Grouping by a Label](https://docs.sqlalchemy.org/en/14/tutorial/data\_select.html#tutorial-order-by-label)) — имена меток, которые мы создаем, также могут упоминаться в предложении **ORDER BY** или **GROUP BY** в [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select).
{% endhint %}

### Выбор с помощью выражений текстового столбца

Когда мы создаем объект [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select) с помощью функции [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), мы обычно передаем ему серию объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) и [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), которые были определены с использованием [метаданных таблицы](https://docs.sqlalchemy.org/en/14/tutorial/metadata.html#tutorial-working-with-metadata), или при использовании ORM мы можем отправлять атрибуты, отображаемые ORM, которые представляют столбцы таблицы. Однако иногда также необходимо создавать произвольные блоки SQL внутри операторов, такие как константные строковые выражения, или просто некоторый произвольный SQL, который быстрее написать буквально.

Конструкция [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text), представленная в разделе «[Работа с транзакциями и DBAPI](../rabota-s-tranzakciyami-i-dbapi.md)», на самом деле может быть встроена непосредственно в конструкцию [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), например, как показано ниже, где мы создаем жестко запрограммированный строковый литерал «**some label**» и встраиваем его в оператор **SELECT**:

```python
>>> from sqlalchemy import text
>>> stmt = (
...     select(
...         text("'some phrase'"), user_table.c.name
...     ).order_by(user_table.c.name)
... )
>>> with engine.connect() as conn:
...     print(conn.execute(stmt).all())
```

```sql
BEGIN (implicit)
SELECT 'some phrase', user_account.name
FROM user_account ORDER BY user_account.name
[generated in ...] ()
```

```python
[('some phrase', 'patrick'), ('some phrase', 'sandy'), ('some phrase', 'spongebob')]
```

```sql
ROLLBACK
```

Хотя конструкция [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text) может использоваться в большинстве случаев для ввода буквальных фраз SQL, чаще всего мы имеем дело с текстовыми единицами, каждая из которых представляет отдельное выражение столбца. В этом распространенном случае мы можем получить больше функциональности из нашего текстового фрагмента, используя вместо этого конструкцию [literal\_column()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.literal\_column). Этот объект аналогичен [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text), за исключением того, что вместо произвольного SQL-запроса в любой форме он явно представляет один **«столбец»**, который затем может быть помечен и на него можно ссылаться в подзапросах и других выражениях:

```python
>>> from sqlalchemy import literal_column
>>> stmt = (
...     select(
...         literal_column("'some phrase'").label("p"), user_table.c.name
...     ).order_by(user_table.c.name)
... )
>>> with engine.connect() as conn:
...     for row in conn.execute(stmt):
...         print(f"{row.p}, {row.name}")
```

```sql
BEGIN (implicit)
SELECT 'some phrase' AS p, user_account.name
FROM user_account ORDER BY user_account.name
[generated in ...] ()
```

```python
some phrase, patrick
some phrase, sandy
some phrase, spongebob
```

```sql
ROLLBACK
```

Обратите внимание, что в обоих случаях при использовании [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text) или [literal\_column()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.literal\_column) мы пишем синтаксическое выражение SQL, а не буквальное значение. Поэтому мы должны включить любые кавычки или синтаксис, необходимые для SQL, который мы хотим увидеть.

## Условие WHERE

SQLAlchemy позволяет нам составлять выражения SQL, такие как `name = 'squidward'` или `user_id > 10`, используя стандартные операторы Python в сочетании с [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) и подобными объектами. Для логических выражений большинство операторов Python, таких как `==`, `!=`, `<`, `>=` и т. д., генерируют новые объекты SQL Expression, а не простые логические значения `True/False`:

```python
>>> print(user_table.c.name == 'squidward')
user_account.name = :name_1

>>> print(address_table.c.user_id > 10)
address.user_id > :user_id_1
```

Мы можем использовать подобные выражения для создания предложения **WHERE**, передавая полученные объекты методу [Select.where()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.where):

```python
>>> print(select(user_table).where(user_table.c.name == 'squidward'))
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = :name_1
```

Чтобы создать несколько выражений, объединенных оператором **AND**, метод [Select.where()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.where) может вызываться любое количество раз:

```python
>>> print(
...     select(address_table.c.email_address).
...     where(user_table.c.name == 'squidward').
...     where(address_table.c.user_id == user_table.c.id)
... )
```

```sql
SELECT address.email_address
FROM address, user_account
WHERE user_account.name = :name_1 AND address.user_id = user_account.id
```

Один вызов [Select.where()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.where) также принимает несколько выражений с тем же эффектом:

```python
>>> print(
...     select(address_table.c.email_address).
...     where(
...          user_table.c.name == 'squidward',
...          address_table.c.user_id == user_table.c.id
...     )
... )
```

```sql
SELECT address.email_address
FROM address, user_account
WHERE user_account.name = :name_1 AND address.user_id = user_account.id
```

Сочетания «**AND**» и «**OR**» доступны напрямую с помощью функций [and()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.and\_) и [or()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.or\_), как показано ниже в терминах объектов ORM:

```python
>>> from sqlalchemy import and_, or_
>>> print(
...     select(Address.email_address).
...     where(
...         and_(
...             or_(User.name == 'squidward', User.name == 'sandy'),
...             Address.user_id == User.id
...         )
...     )
... )
```

```sql
SELECT address.email_address
FROM address, user_account
WHERE (user_account.name = :name_1 OR user_account.name = :name_2)
AND address.user_id = user_account.id
```

Для простых сравнений «на равенство» с одним объектом также есть популярный метод, известный как [Select.filter\_by()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.filter\_by), который принимает аргументы ключевого слова, соответствующие ключам столбцов или именам атрибутов ORM. Он будет фильтровать по крайнему левому предложению **FROM** или последнему присоединенному объекту:

```python
>>> print(
...     select(User).filter_by(name='spongebob', fullname='Spongebob Squarepants')
... )
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = :name_1 AND user_account.fullname = :fullname_1
```

{% hint style="info" %}
Смотри также:

[Operator Reference](https://docs.sqlalchemy.org/en/14/core/operators.html) — описания большинства функций операторов SQL в SQLAlchemy.
{% endhint %}

## Явные условия FROM и JOIN

Как упоминалось ранее, предложение **FROM** обычно **выводится** на основе выражений, которые мы устанавливаем в предложении столбцов, а также других элементах [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select).

Если мы устанавливаем один столбец из определенной таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) в предложении **COLUMNS**, это также помещает эту таблицу в предложение **FROM**:

```python
>>> print(select(user_table.c.name))
```

```sql
SELECT user_account.name
FROM user_account
```

Если бы мы поместили столбцы из двух таблиц, то получили бы предложение **FROM**, разделенное запятыми:

```python
>>> print(select(user_table.c.name, address_table.c.email_address))
```

```sql
SELECT user_account.name, address.email_address
FROM user_account, address
```

Чтобы **JOIN** объединил эти две таблицы вместе, мы обычно используем один из двух методов в [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select). Первый — это метод [Select.join\_from()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join\_from), который позволяет нам явно указать _**левую и правую**_ стороны **JOIN**:

```python
>>> print(
...     select(user_table.c.name, address_table.c.email_address).
...     join_from(user_table, address_table)
... )
```

```sql
SELECT user_account.name, address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
```

Другим является метод [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join), который указывает _**только правую**_ часть **JOIN**, левая сторона выводится:

```python
>>> print(
...     select(user_table.c.name, address_table.c.email_address).
...     join(address_table)
... )
```

```sql
SELECT user_account.name, address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
```

{% hint style="info" %}
**Предложение ON выводится**

При использовании [Select.join\_from()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join\_from) или [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join) мы можем заметить, что предложение **ON** соединения также выводится для нас в случаях простого внешнего ключа. Подробнее об этом в следующем разделе.
{% endhint %}

У нас также есть возможность явно добавлять элементы в предложение **FROM**, если это не выводится так, как мы хотим, из предложения столбцов. Для этого мы используем метод [Select.select\_from()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.select\_from), как показано ниже, где мы устанавливаем **user\_table** в качестве первого элемента в предложении **FROM** и [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join) для установки **address\_table** в качестве второго:

```python
>>> print(
...     select(address_table.c.email_address).
...     select_from(user_table).join(address_table)
... )
```

```sql
SELECT address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
```

Другой пример, когда мы можем захотеть использовать [Select.select\_from()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.select\_from), — это если в нашем предложении столбцов недостаточно информации для предоставления предложения **FROM**. Например, для **SELECT** из общего выражения SQL `count(*)` мы используем элемент SQLAlchemy, известный как [sqlalchemy.sql.expression.func](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.func), для создания функции SQL `count()`:

```python
>>> from sqlalchemy import func
>>> print (
...     select(func.count('*')).select_from(user_table)
... )
```

```sql
SELECT count(:count_2) AS count_1
FROM user_account
```

{% hint style="info" %}
Смотри также:

[Controlling what to Join From](https://docs.sqlalchemy.org/en/14/orm/queryguide.html#orm-queryguide-select-from) — в [ORM Querying Guide](https://docs.sqlalchemy.org/en/14/orm/queryguide.html) — содержит дополнительные примеры и примечания, касающиеся взаимодействия [Select.select\_from()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.select\_from) и [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join).
{% endhint %}

### Установка предложения ON

Предыдущие примеры **JOIN** показали, что конструкция [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select) может соединяться между двумя таблицами и автоматически создавать предложение **ON**. Это происходит в этих примерах, потому что объекты таблицы **user\_table** и **address\_table** включают одно определение [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint), которое используется для формирования этого предложения **ON**.

Если левая и правая цели соединения не имеют такого ограничения или есть несколько ограничений, нам нужно напрямую указать предложение **ON**. И [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join), и [Select.join\_from()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join\_from) принимают дополнительный аргумент для предложения **ON**, который задается с использованием той же механики SQL-выражения, что мы видели в [предложении WHERE](vybor-strok-s-pomoshyu-core-ili-orm.md#uslovie-where):

```python
>>> print(
...     select(address_table.c.email_address).
...     select_from(user_table).
...     join(address_table, user_table.c.id == address_table.c.user_id)
... )
```

```sql
SELECT address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
```

{% hint style="success" %}
**Совет для ORM** — есть еще один способ сгенерировать предложение **ON** при использовании сущностей ORM, которые используют конструкцию отношения [relationship()](../../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), например сопоставление, настроенное в предыдущем разделе в разделе «[Объявление сопоставленных классов](../rabota-s-metadannymi-bazy-dannykh.md#obyavlenie-sopostavlennykh-mapped-klassov)». Это целая тема сама по себе, которая подробно представлена в разделе «[Использование отношений для соединения](../rabota-so-svyazannymi-obektami.md#ispolzovanie-otnoshenii-relationship-v-join)».
{% endhint %}

### OUTER и FULL соединения

Оба метода [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join) и [Select.join\_from()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join\_from) принимают аргументы ключевых слов [Select.join.isouter](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join.params.isouter) и [Select.join.full](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join.params.full), которые отображают **LEFT OUTER JOIN** и **FULL OUTER JOIN** соответственно:

```python
>>> print(
...     select(user_table).join(address_table, isouter=True)
... )
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account LEFT OUTER JOIN address ON user_account.id = address.user_id
```

```python
>>> print(
...     select(user_table).join(address_table, full=True)
... )
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account FULL OUTER JOIN address ON user_account.id = address.user_id
```

Существует также метод [Select.outerjoin()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.outerjoin), эквивалентный использованию `.join(..., isouter=True)`.

{% hint style="info" %}
**Совет**:

SQL также имеет «**RIGHT OUTER JOIN**». SQLAlchemy не отображает это напрямую; вместо этого измените порядок таблиц и используйте «**LEFT OUTER JOIN**».
{% endhint %}

## ORDER BY, GROUP BY, HAVING

Оператор **SELECT** SQL включает предложение **ORDER BY**, которое используется для возврата выбранных строк в заданном порядке.

Предложение **GROUP BY** построено аналогично предложению **ORDER BY** и предназначено для разделения выбранных строк на определенные группы, для которых могут быть вызваны агрегатные функции. Предложение **HAVING** обычно используется с **GROUP BY** и по форме аналогично предложению **WHERE**, за исключением того, что оно применяется к агрегированным функциям, используемым внутри групп.

### ORDER BY

Предложение **ORDER BY** построено в терминах конструкций выражений SQL, обычно основанных на [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) или подобных объектах. Метод [Select.order\_by()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.order\_by) принимает одно или несколько из этих выражений позиционно:

```python
>>> print(select(user_table).order_by(user_table.c.name))
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account ORDER BY user_account.name
```

Сортировка по возрастанию/по убыванию доступна из модификаторов [ColumnElement.asc()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnElement.asc) и [ColumnElement.desc()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnElement.desc), которые также присутствуют в атрибутах, связанных с ORM:

```python
>>> print(select(User).order_by(User.fullname.desc()))
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account ORDER BY user_account.fullname DESC
```

Приведенный выше оператор даст строки, отсортированные по столбцу **user\_account.fullname** в порядке убывания.

### Агрегатные функции с GROUP BY / HAVING

В SQL агрегатные функции позволяют объединять выражения столбцов в нескольких строках для получения единого результата. Примеры включают подсчет, вычисление средних значений, а также определение максимального или минимального значения в наборе значений.

SQLAlchemy предоставляет функции SQL открытым способом, используя пространство имен, известное как [func](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.func). Это специальный объект-конструктор, который будет создавать новые экземпляры [Function](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.Function) при задании имени конкретной функции SQL, которая может иметь любое имя, а также ноль или более аргументов для передачи функции, которые, как и во всех других случаях, это конструкции выражения SQL. Например, чтобы отобразить функцию SQL **COUNT()** для столбца **user\_account.id**, мы вызываем имя **count()**:

```python
>>> from sqlalchemy import func
>>> count_fn = func.count(user_table.c.id)
>>> print(count_fn)
```

```sql
count(user_account.id)
```

Функции SQL более подробно описаны далее в этом руководстве в разделе [Работа с функциями SQL](vybor-strok-s-pomoshyu-core-ili-orm.md#rabota-s-funkciyami-sql).

При использовании агрегатных функций в SQL предложение **GROUP BY** необходимо, поскольку оно позволяет разбивать строки на группы, где агрегатные функции будут применяться к каждой группе отдельно. При запросе неагрегированных столбцов в предложении **COLUMNS** оператора **SELECT** SQL требует, чтобы все эти столбцы подчинялись предложению **GROUP BY**, прямо или косвенно на основе ассоциации первичного ключа. Затем предложение **HAVING** используется аналогично предложению **WHERE**, за исключением того, что оно отфильтровывает строки на основе агрегированных значений, а не прямого содержимого строки.

SQLAlchemy предоставляет эти два предложения с помощью методов [Select.group\_by()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.group\_by) и [Select.having()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.having). Ниже мы иллюстрируем выбор полей имени пользователя, а также количество адресов для тех пользователей, у которых есть более одного адреса:

```python
>>> with engine.connect() as conn:
...     result = conn.execute(
...         select(User.name, func.count(Address.id).label("count")).
...         join(Address).
...         group_by(User.name).
...         having(func.count(Address.id) > 1)
...     )
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT user_account.name, count(address.id) AS count
FROM user_account JOIN address ON user_account.id = address.user_id GROUP BY user_account.name
HAVING count(address.id) > ?
[...] (1,)
```

```python
[('sandy', 2)]
```

```sql
ROLLBACK
```

### Упорядочивание или группировка по метке (label)

Важным приемом, в частности в некоторых базах данных, является возможность **ORDER BY** или **GROUP BY** для выражения, которое уже указано в предложении столбцов, без повторного определения выражения в предложении **ORDER BY** или **GROUP BY** и вместо этого используется имя столбца или имя метки из предложения **COLUMNS**. Эта форма доступна путем передачи строкового текста имени в метод [Select.order\_by()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.order\_by) или [Select.group\_by()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.group\_by). Переданный текст **не отображается напрямую**; вместо этого имя, присвоенное выражению в предложении столбцов и отображаемое как имя этого выражения в контексте, вызывает ошибку, если совпадение не найдено. Унарные модификаторы [asc()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.asc) и [desc()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.desc) также могут использоваться в такой форме:

```python
>>> from sqlalchemy import func, desc
>>> stmt = select(
...         Address.user_id,
...         func.count(Address.id).label('num_addresses')).\
...         group_by("user_id").order_by("user_id", desc("num_addresses"))
>>> print(stmt)
```

```sql
SELECT address.user_id, count(address.id) AS num_addresses
FROM address GROUP BY address.user_id ORDER BY address.user_id, num_addresses DESC
```

## Использование псевдонимов (Alias)

Теперь, когда мы выбираем из нескольких таблиц и используем соединения, мы быстро сталкиваемся со случаем, когда нам нужно несколько раз ссылаться на одну и ту же таблицу в предложении **FROM** оператора. Мы достигаем этого с помощью псевдонимов SQL (**aliases**), которые представляют собой синтаксис, предоставляющий альтернативное имя таблице или подзапросу, из которого на него можно ссылаться в операторе.

В языке выражений SQLAlchemy эти «имена» вместо этого представлены объектами [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause), известными как конструкция [Alias](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Alias), которая создается в Core с использованием метода [FromClause.alias()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause.alias). Конструкция [Alias](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Alias) аналогична конструкции **Table** в том, что она также имеет пространство имен объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) в коллекции **Alias.c**. Например, приведенный ниже оператор **SELECT** возвращает все уникальные пары имен пользователей:

```python
>>> user_alias_1 = user_table.alias()
>>> user_alias_2 = user_table.alias()
>>> print(
...     select(user_alias_1.c.name, user_alias_2.c.name).
...     join_from(user_alias_1, user_alias_2, user_alias_1.c.id > user_alias_2.c.id)
... )
```

```sql
SELECT user_account_1.name, user_account_2.name AS name_1
FROM user_account AS user_account_1
JOIN user_account AS user_account_2 ON user_account_1.id > user_account_2.id
```

### Псевдонимы объектов ORM

ORM-эквивалентом метода [FromClause.alias()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause.alias) является функция ORM [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased), которую можно применять к таким объектам, как **User** и **Address**. Это создает внутренний объект [Alias](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Alias), который соответствует исходному сопоставленному объекту [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), сохраняя при этом функциональность ORM. **SELECT** ниже выбирает из сущности **User** все объекты, которые включают два конкретных адреса электронной почты:

```python
>>> from sqlalchemy.orm import aliased
>>> address_alias_1 = aliased(Address)
>>> address_alias_2 = aliased(Address)
>>> print(
...     select(User).
...     join_from(User, address_alias_1).
...     where(address_alias_1.email_address == 'patrick@aol.com').
...     join_from(User, address_alias_2).
...     where(address_alias_2.email_address == 'patrick@gmail.com')
... )
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
JOIN address AS address_1 ON user_account.id = address_1.user_id
JOIN address AS address_2 ON user_account.id = address_2.user_id
WHERE address_1.email_address = :email_address_1
AND address_2.email_address = :email_address_2
```

{% hint style="info" %}
**Совет**:

Как упоминалось в разделе «[Установка предложения ON](vybor-strok-s-pomoshyu-core-ili-orm.md#ustanovka-predlozheniya-on)», ORM предоставляет другой способ соединения с использованием конструкции [relationship()](../../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for). Приведенный выше пример с использованием псевдонимов демонстрируется с использованием отношения [relationship()](../../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) при [соединении между целями с псевдонимами](../rabota-so-svyazannymi-obektami.md#obedinenie-mezhdu-celyami-s-psevdonimami).
{% endhint %}

## Подзапросы (subquery) и CTE

Подзапрос в SQL — это инструкция **SELECT**, которая отображается в круглых скобках и помещается в контекст заключающей инструкции, обычно это инструкция **SELECT**, но не обязательно.

В этом разделе рассматривается так называемый «нескалярный» подзапрос, который обычно размещается в предложении **FROM** включающего **SELECT**. Мы также рассмотрим **Common Table Expression** или **CTE**, который используется аналогично подзапросу, но включает в себя дополнительные функции.

SQLAlchemy использует объект [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery) для представления подзапроса и [CTE](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CTE) для представления **CTE**, обычно получаемого из методов [Select.subquery()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.subquery) и [Select.cte()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.cte) соответственно. Любой объект может использоваться как элемент **FROM** внутри более крупной конструкции [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select).

Мы можем создать подзапрос [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery), который будет выбирать совокупное количество строк из таблицы адресов (агрегатные функции и **GROUP BY** были представлены ранее в [агрегатных функциях с GROUP BY/HAVING](vybor-strok-s-pomoshyu-core-ili-orm.md#agregatnye-funkcii-s-group-by-having)):

```python
>>> subq = select(
...     func.count(address_table.c.id).label("count"),
...     address_table.c.user_id
... ).group_by(address_table.c.user_id).subquery()
```

Строковое преобразование подзапроса в строку без его встраивания в другой оператор [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select) или другой оператор дает простой оператор **SELECT** без каких-либо закрывающих скобок:

```python
>>> print(subq)
```

```sql
SELECT count(address.id) AS count, address.user_id
FROM address GROUP BY address.user_id
```

Объект [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery) ведет себя так же, как и любой другой объект **FROM**, такой как таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), в частности, он включает в себя пространство имен **Subquery.c** столбцов, которые он выбирает. Мы можем использовать это пространство имен для ссылки как на столбец **user\_id**, так и на наше пользовательское помеченное выражение **count**:

```python
>>> print(select(subq.c.user_id, subq.c.count))
```

```sql
SELECT anon_1.user_id, anon_1.count
FROM (SELECT count(address.id) AS count, address.user_id AS user_id
FROM address GROUP BY address.user_id) AS anon_1
```

С выбором строк, содержащихся в объекте **subq**, мы можем применить объект к большему [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), который присоединит данные к таблице **user\_account**:

```python
>>> stmt = select(
...    user_table.c.name,
...    user_table.c.fullname,
...    subq.c.count
... ).join_from(user_table, subq)

>>> print(stmt)
```

```sql
SELECT user_account.name, user_account.fullname, anon_1.count
FROM user_account JOIN (SELECT count(address.id) AS count, address.user_id AS user_id
FROM address GROUP BY address.user_id) AS anon_1 ON user_account.id = anon_1.user_id
```

Чтобы присоединиться от **user\_account** к адресу **address**, мы использовали метод [Select.join\_from()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join\_from). Как было показано ранее, предложение **ON** этого соединения снова **было выведено** на основе ограничений внешнего ключа. Несмотря на то, что подзапрос SQL сам по себе не имеет никаких ограничений, SQLAlchemy может воздействовать на ограничения, представленные в столбцах, определяя, что столбец **subq.c.user\_id** является **производным** от столбца **address\_table.c.user\_id**, который действительно выражает обратную связь внешнего ключа отношением в столбец **user\_table.c.id**, который затем используется для генерации предложения **ON**.

### Общие табличные выражения (CTE)

Использование конструкции [CTE](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CTE) в SQLAlchemy практически не отличается от использования конструкции [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery). Изменив вызов метода [Select.subquery()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.subquery) на использование вместо него [Select.cte()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.cte), мы можем таким же образом использовать результирующий объект в качестве элемента **FROM**, но отображаемый SQL представляет собой совершенно другой синтаксис общего табличного выражения:

```python
>>> subq = select(
...     func.count(address_table.c.id).label("count"),
...     address_table.c.user_id
... ).group_by(address_table.c.user_id).cte()

>>> stmt = select(
...    user_table.c.name,
...    user_table.c.fullname,
...    subq.c.count
... ).join_from(user_table, subq)

>>> print(stmt)
```

```sql
WITH anon_1 AS
(SELECT count(address.id) AS count, address.user_id AS user_id
FROM address GROUP BY address.user_id)
 SELECT user_account.name, user_account.fullname, anon_1.count
FROM user_account JOIN anon_1 ON user_account.id = anon_1.user_id
```

Конструкция [CTE](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CTE) также может использоваться в «рекурсивном» стиле и в более сложных случаях может быть составлена из предложения **RETURNING** оператора **INSERT**, **UPDATE** или **DELETE**. Строка документации для [CTE](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CTE) содержит подробную информацию об этих дополнительных шаблонах.

В обоих случаях подзапрос и **CTE** были названы на уровне SQL с использованием «**анонимного**» имени. В коде Python нам вообще не нужно указывать эти имена. Идентификатор объекта экземпляров [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery) или [CTE](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CTE) служит синтаксическим идентификатором объекта при отображении. Имя, которое будет отображено в SQL, можно указать, передав его в качестве первого аргумента методов [Select.subquery()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.subquery) или [Select.cte()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.cte).

{% hint style="info" %}
Смотри также:

[Select.subquery()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.subquery) — дополнительная информация о подзапросах

[Select.cte()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.cte) — примеры для **CTE**, включая использование **RECURSIVE**, а также **CTE**, ориентированных на **DML**.
{% endhint %}

### Подзапросы/CTE сущностей ORM

В ORM конструкция [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) может использоваться для связывания объекта ORM, такого как наш класс **User** или **Address**, с любым понятием [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause), представляющим источник строк. В предыдущем разделе «[Псевдонимы сущностей ORM](vybor-strok-s-pomoshyu-core-ili-orm.md#psevdonimy-obektov-orm)» показано использование [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) для связывания сопоставленного класса с псевдонимом [Alias](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Alias) его сопоставленной таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table). Здесь мы иллюстрируем, как [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) делает одно и то же как для подзапроса [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery), так и для [CTE](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CTE), сгенерированного для конструкции [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), которая в конечном итоге является производной от той же сопоставленной таблицы [Table](vybor-strok-s-pomoshyu-core-ili-orm.md#podzaprosy-subquery-i-cte).

Ниже приведен пример применения [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) к конструкции [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery), чтобы объекты ORM можно было извлечь из ее строк. Результат показывает серию объектов **User** и **Address**, где данные для каждого объекта **Address** в конечном итоге получены из подзапроса к таблице **address**, а не напрямую к этой таблице:

```python
>>> subq = select(Address).where(~Address.email_address.like('%@aol.com')).subquery()
>>> address_subq = aliased(Address, subq)
>>> stmt = select(User, address_subq).join_from(User, address_subq).order_by(User.id, address_subq.id)
>>> with Session(engine) as session:
...     for user, address in session.execute(stmt):
...         print(f"{user} {address}")
```

```sql
BEGIN (implicit)
SELECT user_account.id, user_account.name, user_account.fullname,
anon_1.id AS id_1, anon_1.email_address, anon_1.user_id
FROM user_account JOIN
(SELECT address.id AS id, address.email_address AS email_address, address.user_id AS user_id
FROM address
WHERE address.email_address NOT LIKE ?) AS anon_1 ON user_account.id = anon_1.user_id
ORDER BY user_account.id, anon_1.id
[...] ('%@aol.com',)
```

```python
User(id=1, name='spongebob', fullname='Spongebob Squarepants') Address(id=1, email_address='spongebob@sqlalchemy.org')
User(id=2, name='sandy', fullname='Sandy Cheeks') Address(id=2, email_address='sandy@sqlalchemy.org')
User(id=2, name='sandy', fullname='Sandy Cheeks') Address(id=3, email_address='sandy@squirrelpower.org')
```

```sql
ROLLBACK
```

Далее следует еще один пример, который точно такой же, за исключением того, что вместо него используется конструкция [CTE](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CTE):

```python
>>> cte_obj = select(Address).where(~Address.email_address.like('%@aol.com')).cte()
>>> address_cte = aliased(Address, cte_obj)
>>> stmt = select(User, address_cte).join_from(User, address_cte).order_by(User.id, address_cte.id)
>>> with Session(engine) as session:
...     for user, address in session.execute(stmt):
...         print(f"{user} {address}")
```

```sql
BEGIN (implicit)
WITH anon_1 AS
(SELECT address.id AS id, address.email_address AS email_address, address.user_id AS user_id
FROM address
WHERE address.email_address NOT LIKE ?)
SELECT user_account.id, user_account.name, user_account.fullname,
anon_1.id AS id_1, anon_1.email_address, anon_1.user_id
FROM user_account
JOIN anon_1 ON user_account.id = anon_1.user_id
ORDER BY user_account.id, anon_1.id
[...] ('%@aol.com',)
```

```python
User(id=1, name='spongebob', fullname='Spongebob Squarepants') Address(id=1, email_address='spongebob@sqlalchemy.org')
User(id=2, name='sandy', fullname='Sandy Cheeks') Address(id=2, email_address='sandy@sqlalchemy.org')
User(id=2, name='sandy', fullname='Sandy Cheeks') Address(id=3, email_address='sandy@squirrelpower.org')
```

```sql
ROLLBACK
```

{% hint style="info" %}
Смотри также:

[Selecting Entities from Subqueries](https://docs.sqlalchemy.org/en/14/orm/queryguide.html#orm-queryguide-subqueries) - в документации [ORM Querying Guide](https://docs.sqlalchemy.org/en/14/orm/queryguide.html)
{% endhint %}

## Скалярные и коррелированные подзапросы

**Скалярный** подзапрос — это подзапрос, который _**возвращает ровно ноль или одну строку и ровно один столбец**_. Затем подзапрос используется в предложении **COLUMNS** или **WHERE** объемлющего оператора **SELECT** и отличается от обычного подзапроса тем, что _**не используется**_ в предложении **FROM**. [**Коррелированный** подзапрос](https://docs.sqlalchemy.org/en/14/glossary.html#term-correlated-subquery) — это скалярный подзапрос, который _**ссылается на таблицу**_ во включающем операторе **SELECT**.

SQLAlchemy представляет скалярный подзапрос с помощью конструкции [ScalarSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.ScalarSelect), которая является частью иерархии выражений [ColumnElement](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnElement'), в отличие от обычного подзапроса, представленного конструкцией [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery), которая находится в иерархии [FromClause](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause).

Скалярные подзапросы часто, но не обязательно, используются с агрегатными функциями, представленными ранее в [агрегатных функциях с GROUP BY/HAVING](vybor-strok-s-pomoshyu-core-ili-orm.md#agregatnye-funkcii-s-group-by-having). Скалярный подзапрос явно указывается с помощью метода [Select.scalar\_subquery()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.scalar\_subquery), как показано ниже. Это строковая форма по умолчанию, когда она сама по себе представляет собой обычный оператор **SELECT**, который выбирает из двух таблиц:

```python
>>> subq = select(func.count(address_table.c.id)).\
...             where(user_table.c.id == address_table.c.user_id).\
...             scalar_subquery()
>>> print(subq)
```

```sql
(SELECT count(address.id) AS count_1
FROM address, user_account
WHERE user_account.id = address.user_id)
```

Приведенный выше объект **subq** теперь попадает в иерархию SQL-выражений [ColumnElement](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnElement), поскольку его можно использовать как любое другое выражение столбца:

```python
>>> print(subq == 5)
```

```sql
(SELECT count(address.id) AS count_1
FROM address, user_account
WHERE user_account.id = address.user_id) = :param_1
```

Хотя скалярный подзапрос сам по себе отображает как **user\_account**, так и **address** в своем предложении **FROM**, когда он преобразован в строку, при встраивании его во включающую конструкцию [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), которая имеет дело с таблицей **user\_account**, таблица **user\_account** автоматически **коррелируется**, то есть она не отображается в предложение **FROM** подзапроса:

```python
>>> stmt = select(user_table.c.name, subq.label("address_count"))
>>> print(stmt)
```

```sql
SELECT user_account.name, (SELECT count(address.id) AS count_1
FROM address
WHERE user_account.id = address.user_id) AS address_count
FROM user_account
```

Простые коррелированные подзапросы обычно делают то, что нужно. Однако в случае неоднозначной корреляции SQLAlchemy сообщит нам, что требуется больше ясности:

```python
>>> stmt = select(
...     user_table.c.name,
...     address_table.c.email_address,
...     subq.label("address_count")
... ).\
... join_from(user_table, address_table).\
... order_by(user_table.c.id, address_table.c.id)
>>> print(stmt)
Traceback (most recent call last):
...
InvalidRequestError: Select statement '<... Select object at ...>' returned
no FROM clauses due to auto-correlation; specify correlate(<tables>) to
control correlation manually.
```

Чтобы указать, что **user\_table** является той таблицей, которую мы пытаемся скоррелировать, мы указываем это с помощью методов [ScalarSelect.correlate()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.ScalarSelect.correlate) или [ScalarSelect.correlate\_except()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.ScalarSelect.correlate\_except):

```python
>>> subq = select(func.count(address_table.c.id)).\
...             where(user_table.c.id == address_table.c.user_id).\
...             scalar_subquery().correlate(user_table)
```

Затем оператор может вернуть данные для этого столбца, как и для любого другого:

```python
>>> with engine.connect() as conn:
...     result = conn.execute(
...         select(
...             user_table.c.name,
...             address_table.c.email_address,
...             subq.label("address_count")
...         ).
...         join_from(user_table, address_table).
...         order_by(user_table.c.id, address_table.c.id)
...     )
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT user_account.name, address.email_address, (SELECT count(address.id) AS count_1
FROM address
WHERE user_account.id = address.user_id) AS address_count
FROM user_account JOIN address ON user_account.id = address.user_id ORDER BY user_account.id, address.id
[...] ()
```

```python
[('spongebob', 'spongebob@sqlalchemy.org', 1),
 ('sandy', 'sandy@sqlalchemy.org', 2),
 ('sandy', 'sandy@squirrelpower.org', 2)]
```

```sql
ROLLBACK
```

## UNION, UNION ALL и другие операции над множествами

В SQL операторы **SELECT** могут быть объединены вместе с помощью SQL-операции **UNION** или **UNION ALL**, которая создает набор всех строк, созданных одним или несколькими операторами вместе. Также возможны другие операции с наборами, такие как **INTERSECT \[ALL]** и **EXCEPT \[ALL]**.

Конструкция SQLAlchemy [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select) поддерживает композиции такого рода с использованием таких функций, как [union()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.union), [intersect()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.intersect) и [except\_()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.except\_), а также **«all»** аналогов [union\_all()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.union\_all), [intersect\_all()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.intersect\_all) и [except\_all()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.except\_all). Все эти функции принимают произвольное количество вложенных элементов выбора (_selectable_), которые обычно являются конструкциями [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), но также могут быть существующей композицией.

Конструкцией, создаваемой этими функциями, является [CompoundSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CompoundSelect), которая используется так же, как конструкция [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), за исключением того, что у нее меньше методов. Например, [CompoundSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CompoundSelect), созданный [union\_all()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.union\_all), может быть вызван непосредственно с помощью [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Connection.execute):

```python
>>> from sqlalchemy import union_all
>>> stmt1 = select(user_table).where(user_table.c.name == 'sandy')
>>> stmt2 = select(user_table).where(user_table.c.name == 'spongebob')
>>> u = union_all(stmt1, stmt2)
>>> with engine.connect() as conn:
...     result = conn.execute(u)
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = ?
UNION ALL SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = ?
[generated in ...] ('sandy', 'spongebob')
```

```python
[(2, 'sandy', 'Sandy Cheeks'), (1, 'spongebob', 'Spongebob Squarepants')]
```

```sql
ROLLBACK
```

Чтобы использовать [CompoundSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CompoundSelect) в качестве подзапроса, как и в случае с [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), он предоставляет метод [SelectBase.subquery()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.SelectBase.subquery), который создаст объект [Subquery](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Subquery) с коллекцией [FromClause.c](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.FromClause.c), на который можно ссылаться во включающем [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select):

```python
>>> u_subq = u.subquery()
>>> stmt = (
...     select(u_subq.c.name, address_table.c.email_address).
...     join_from(address_table, u_subq).
...     order_by(u_subq.c.name, address_table.c.email_address)
... )
>>> with engine.connect() as conn:
...     result = conn.execute(stmt)
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT anon_1.name, address.email_address
FROM address JOIN
  (SELECT user_account.id AS id, user_account.name AS name, user_account.fullname AS fullname
  FROM user_account
  WHERE user_account.name = ?
UNION ALL
  SELECT user_account.id AS id, user_account.name AS name, user_account.fullname AS fullname
  FROM user_account
  WHERE user_account.name = ?)
AS anon_1 ON anon_1.id = address.user_id
ORDER BY anon_1.name, address.email_address
[generated in ...] ('sandy', 'spongebob')
```

```python
[('sandy', 'sandy@sqlalchemy.org'),
 ('sandy', 'sandy@squirrelpower.org'),
 ('spongebob', 'spongebob@sqlalchemy.org')]
```

```sql
ROLLBACK
```

### Выбор объектов ORM из объединений (UNION)

В предыдущих примерах показано, как построить **UNION** для двух объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), чтобы затем вернуть строки базы данных. Если мы хотим использовать **UNION** или другую операцию набора для выбора строк, которые мы затем получаем как объекты ORM, есть два подхода, которые можно использовать. В обоих случаях мы сначала создаем объект [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select) или [CompoundSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CompoundSelect), представляющий оператор **SELECT/UNION/etc**, который мы хотим выполнить; этот оператор должен быть составлен для целевых объектов ORM или их базовых сопоставленных объектов [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table):

```python
>>> stmt1 = select(User).where(User.name == 'sandy')
>>> stmt2 = select(User).where(User.name == 'spongebob')
>>> u = union_all(stmt1, stmt2)
```

Для простого **SELECT** с **UNION**, который еще не вложен в подзапрос, их часто можно использовать в контексте выборки объекта ORM с помощью метода [Select.from\_statement()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.from\_statement). При таком подходе оператор **UNION** представляет весь запрос; никакие дополнительные критерии не могут быть добавлены после использования [Select.from\_statement()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.from\_statement):

```python
>>> orm_stmt = select(User).from_statement(u)
>>> with Session(engine) as session:
...     for obj in session.execute(orm_stmt).scalars():
...         print(obj)
```

```sql
BEGIN (implicit)
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = ? UNION ALL SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = ?
[generated in ...] ('sandy', 'spongebob')
```

```python
User(id=2, name='sandy', fullname='Sandy Cheeks')
User(id=1, name='spongebob', fullname='Spongebob Squarepants')
```

```sql
ROLLBACK
```

Чтобы использовать **UNION** или другую конструкцию, связанную с набором, в качестве компонента, связанного с сущностью, более гибким образом, конструкция [CompoundSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CompoundSelect) может быть организована в подзапрос с использованием [CompoundSelect.subquery()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.CompoundSelect.subquery), который затем связывается с объектами ORM с помощью функции [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased). Это работает так же, как представлено в [ORM Entity Subqueries/CTEs](vybor-strok-s-pomoshyu-core-ili-orm.md#podzaprosy-cte-sushnostei-orm), чтобы сначала создать специальное «сопоставление» (_mapping_) нашего желаемого объекта с подзапросом, а затем выбрать из него этот новый объект, как если бы это был любой другой сопоставленный класс. В приведенном ниже примере мы можем добавить дополнительные критерии, такие как **ORDER BY**, вне самого **UNION**, поскольку мы можем фильтровать или упорядочивать столбцы, экспортируемые подзапросом:

```python
>>> user_alias = aliased(User, u.subquery())
>>> orm_stmt = select(user_alias).order_by(user_alias.id)
>>> with Session(engine) as session:
...     for obj in session.execute(orm_stmt).scalars():
...         print(obj)
```

```sql
BEGIN (implicit)
SELECT anon_1.id, anon_1.name, anon_1.fullname
FROM (SELECT user_account.id AS id, user_account.name AS name, user_account.fullname AS fullname
FROM user_account
WHERE user_account.name = ? UNION ALL SELECT user_account.id AS id, user_account.name AS name, user_account.fullname AS fullname
FROM user_account
WHERE user_account.name = ?) AS anon_1 ORDER BY anon_1.id
[generated in ...] ('sandy', 'spongebob')
```

```python
User(id=1, name='spongebob', fullname='Spongebob Squarepants')
User(id=2, name='sandy', fullname='Sandy Cheeks')
```

```sql
ROLLBACK
```

{% hint style="info" %}
Смотри также:

[Selecting Entities from UNIONs and other set operations](https://docs.sqlalchemy.org/en/14/orm/queryguide.html#orm-queryguide-unions) - в документации [ORM Querying Guide](https://docs.sqlalchemy.org/en/14/orm/queryguide.html)
{% endhint %}

## Подзапросы EXISTS

Ключевое слово SQL **EXISTS** — это оператор, который используется со [скалярными подзапросами](vybor-strok-s-pomoshyu-core-ili-orm.md#skalyarnye-i-korrelirovannye-podzaprosy) для возврата логического значения **true** или **false** в зависимости от того, будет ли инструкция **SELECT** возвращать строку. SQLAlchemy включает вариант объекта [ScalarSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.ScalarSelect) под названием [Exists](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Exists), который будет генерировать подзапрос **EXISTS** и наиболее удобно генерировать с помощью метода [SelectBase.exists()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.SelectBase.exists). Ниже мы создаем **EXISTS**, чтобы мы могли возвращать строки **user\_account**, которые имеют более одной связанной строки в **address**:

```python
>>> subq = (
...     select(func.count(address_table.c.id)).
...     where(user_table.c.id == address_table.c.user_id).
...     group_by(address_table.c.user_id).
...     having(func.count(address_table.c.id) > 1)
... ).exists()
>>> with engine.connect() as conn:
...     result = conn.execute(
...         select(user_table.c.name).where(subq)
...     )
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT user_account.name
FROM user_account
WHERE EXISTS (SELECT count(address.id) AS count_1
FROM address
WHERE user_account.id = address.user_id GROUP BY address.user_id
HAVING count(address.id) > ?)
[...] (1,)
```

```python
[('sandy',)]
```

```sql
ROLLBACK
```

Конструкция **EXISTS** чаще всего используется как отрицание, например, **NOT EXISTS**, так как обеспечивает эффективную с SQL форму поиска строк, для которых в связанной таблице нет строк. Ниже мы выбираем имена пользователей, у которых нет адресов электронной почты; обратите внимание на двоичный оператор отрицания **`(~)`**, используемый во втором предложении **WHERE**:

```python
>>> subq = (
...     select(address_table.c.id).
...     where(user_table.c.id == address_table.c.user_id)
... ).exists()
>>> with engine.connect() as conn:
...     result = conn.execute(
...         select(user_table.c.name).where(~subq)
...     )
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT user_account.name
FROM user_account
WHERE NOT (EXISTS (SELECT address.id
FROM address
WHERE user_account.id = address.user_id))
[...] ()
```

```python
[('patrick',)]
```

```sql
ROLLBACK
```

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел руководства: [Выбор строк (работа с SQL функциями)](vybor-strok-rabota-s-sql-funkciyami.md)
{% endhint %}
