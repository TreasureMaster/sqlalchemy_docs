# Tutorial часть 2 (insert, update, query)

## Добавление и обновление объектов

Чтобы сохранить наш объект **User**, мы добавим [Session.add()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.add) его в нашу сессию [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session):

```python
>>> ed_user = User(name='ed', fullname='Ed Jones', nickname='edsnickname')
>>> session.add(ed_user)
```

В этот момент мы говорим, что экземпляр находится **в ожидании (pending)**; SQL еще не выдан, и объект еще не представлен строкой в базе данных. Сессия [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) выдаст SQL для сохранения _**Ed Jones**_, как только это потребуется, используя процесс, известный как сброс **flush**. Если мы запросим базу данных для _**Ed Jones**_, вся ожидающая информация будет сначала сброшена, а запрос будет выдан сразу после этого.

Например, ниже мы создаем новый объект [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query), который загружает экземпляры **User**. Мы фильтруем (**filter\_by**) по атрибуту имени _**name**_ ed и указываем, что нам нужен только первый результат в полном списке строк. Возвращается экземпляр **User**, который эквивалентен тому, что мы добавили:

```python
>>> our_user = session.query(User).filter_by(name='ed').first() 
>>> our_user
<User(name='ed', fullname='Ed Jones', nickname='edsnickname')>
```

```sql
BEGIN (implicit)
INSERT INTO users (name, fullname, nickname) VALUES (?, ?, ?)
[...] ('ed', 'Ed Jones', 'edsnickname')
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name = ?
 LIMIT ? OFFSET ?
[...] ('ed', 1, 0)
```

Фактически, сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) определил, что возвращенная строка является **той же** строкой, что и уже представленная в его внутренней карте объектов, поэтому мы фактически получили идентичный экземпляр, который мы только что добавили:

```python
>>> ed_user is our_user
True
```

Работающая здесь концепция ORM известна как карта идентификаторов ([identity map](https://docs.sqlalchemy.org/en/14/glossary.html#term-identity-map)) и гарантирует, что все операции с определенной строкой в Session выполняются с одним и тем же набором данных. Как только объект с определенным первичным ключом присутствует в сеансе, все запросы SQL в этом сеансе всегда будут возвращать один и тот же объект Python для этого конкретного первичного ключа; это также вызовет ошибку, если будет предпринята попытка разместить в Session второй, уже сохраненный объект с тем же первичным ключом.

Мы можем добавить сразу несколько объектов **User**, используя **add\_all()**:

```python
>>> session.add_all([
...     User(name='wendy', fullname='Wendy Williams', nickname='windy'),
...     User(name='mary', fullname='Mary Contrary', nickname='mary'),
...     User(name='fred', fullname='Fred Flintstone', nickname='freddy')])
```

Кроме того, мы решили, что никнейм Эда не очень хорош, поэтому давайте изменим его:

```python
>>> ed_user.nickname = 'eddie'
```

Сессия обращает внимание. Она знает, например, что Ed Jones был изменен:

```python
>>> session.dirty
IdentitySet([<User(name='ed', fullname='Ed Jones', nickname='eddie')>])
```

и что ожидаются три новых объекта **User**:

```python
>>> session.new  
IdentitySet([<User(name='wendy', fullname='Wendy Williams', nickname='windy')>,
<User(name='mary', fullname='Mary Contrary', nickname='mary')>,
<User(name='fred', fullname='Fred Flintstone', nickname='freddy')>])
```

Мы сообщаем сеансу, что хотели бы внести все оставшиеся изменения в базу данных и зафиксировать транзакцию, которая **все это время выполнялась**. Мы делаем это через [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit). Сессия выдает оператор **UPDATE** для изменения псевдонима на «ed», а также операторы **INSERT** для трех новых добавленных нами объектов **User**:

```python
>>> session.commit()
```

```sql
UPDATE users SET nickname=? WHERE users.id = ?
[...] ('eddie', 1)
INSERT INTO users (name, fullname, nickname) VALUES (?, ?, ?)
[...] ('wendy', 'Wendy Williams', 'windy')
INSERT INTO users (name, fullname, nickname) VALUES (?, ?, ?)
[...] ('mary', 'Mary Contrary', 'mary')
INSERT INTO users (name, fullname, nickname) VALUES (?, ?, ?)
[...] ('fred', 'Fred Flintstone', 'freddy')
COMMIT
```

[Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit) сбрасывает оставшиеся изменения в базу данных и фиксирует транзакцию. Ресурсы соединения, на которые ссылается сеанс, теперь возвращаются в пул соединений. Последующие операции с этим сеансом будут происходить **в новой транзакции**, которая снова будет повторно получать ресурсы соединения, когда это необходимо.

Если мы посмотрим на атрибут _**id**_ Эда, который раньше был `None`, теперь он имеет значение:

```python
>>> ed_user.id 
1
```

```sql
BEGIN (implicit)
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.id = ?
[...] (1,)
```

После того, как сеанс вставит новые строки в базу данных, все вновь сгенерированные идентификаторы и значения по умолчанию, созданные базой данных, станут доступными для экземпляра либо сразу, либо через загрузку при первом доступе. В этом случае вся строка была перезагружена при доступе, потому что новая транзакция была начата после вызова [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit). SQLAlchemy по умолчанию обновляет данные из предыдущей транзакции при первом доступе к ним в рамках новой транзакции, чтобы было доступно самое последнее состояние. Уровень перезагрузки настраивается, как описано в разделе [Использование сеанса](https://docs.sqlalchemy.org/en/14/orm/session.html).

{% hint style="info" %}
**Состояние объекта сеанса**

По мере того, как наш объект **User** перемещался из состояния вне сеанса внутрь сеанса без первичного ключа и фактически вставлялся, он перемещался между тремя из пяти доступных «состояний объекта» — переходным **transient**, ожидающим **pending** и постоянным **persistent**. Знать об этих состояниях и о том, что они означают, всегда полезно — обязательно прочтите [Quickie Intro to Object States](https://docs.sqlalchemy.org/en/14/orm/session\_state\_management.html#session-object-states) для краткого обзора.
{% endhint %}

## Откат (Rolling Back)

Поскольку сеанс работает внутри транзакции, мы также можем откатить сделанные изменения. Давайте внесем два изменения, которые мы вернем; Имя пользователя _**ed\_user**_ устанавливается на Edwardo:

```python
>>> ed_user.name = 'Edwardo'
```

и мы добавим еще одного ошибочного пользователя, _**fake\_user**_:

```python
>>> fake_user = User(name='fakeuser', fullname='Invalid', nickname='12345')
>>> session.add(fake_user)
```

Запросив сессию, мы видим, что они сбрасываются в текущую транзакцию:

```python
>>> session.query(User).filter(User.name.in_(['Edwardo', 'fakeuser'])).all()
[<User(name='Edwardo', fullname='Ed Jones', nickname='eddie')>, <User(name='fakeuser', fullname='Invalid', nickname='12345')>]
```

```sql
UPDATE users SET name=? WHERE users.id = ?
[...] ('Edwardo', 1)
INSERT INTO users (name, fullname, nickname) VALUES (?, ?, ?)
[...] ('fakeuser', 'Invalid', '12345')
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name IN (?, ?)
[...] ('Edwardo', 'fakeuser')
```

Откатываясь назад, мы видим, что имя _**ed\_user**_ снова стало ed, а _**fake\_user**_ выкинут из сессии:

```python
>>> session.rollback()
```

```sql
ROLLBACK
```

```python
>>> ed_user.name
u'ed'
>>> fake_user in session
False
```

```sql
BEGIN (implicit)
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.id = ?
[...] (1,)
```

выдача **SELECT** иллюстрирует изменения, внесенные в базу данных:

```python
>>> session.query(User).filter(User.name.in_(['ed', 'fakeuser'])).all()
[<User(name='ed', fullname='Ed Jones', nickname='eddie')>]
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name IN (?, ?)
[...] ('ed', 'fakeuser')
```

## Запрос (Quering)

Объект [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) создается с помощью метода **query()** в [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session). Эта функция принимает переменное количество аргументов, которые могут быть любой комбинацией классов и дескрипторов, оснащенных классами. Ниже мы указываем запрос **Query**, который загружает экземпляры пользователя **User**. При оценке в итеративном контексте возвращается список имеющихся объектов **User**:

```python
>>> for instance in session.query(User).order_by(User.id):
...     print(instance.name, instance.fullname)
ed Ed Jones
wendy Wendy Williams
mary Mary Contrary
fred Fred Flintstone
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users ORDER BY users.id
[...] ()
```

Запрос [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) также принимает в качестве аргументов дескрипторы с инструментами ORM. Каждый раз, когда несколько сущностей класса или сущностей на основе столбцов выражаются в качестве аргументов функции **query()**, возвращаемый результат выражается в виде кортежей:

```python
>>> for name, fullname in session.query(User.name, User.fullname):
...     print(name, fullname)
ed Ed Jones
wendy Wendy Williams
mary Mary Contrary
fred Fred Flintstone
```

```sql
SELECT users.name AS users_name,
        users.fullname AS users_fullname
FROM users
[...] ()
```

Кортежи, возвращаемые [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query), - это _**именованные**_ кортежи, предоставляемыми классом [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row), и с ними можно обращаться так же, как с обычным объектом Python. Имена такие же, как имя атрибута для атрибута и имя класса для класса:

```python
>>> for row in session.query(User, User.name).all():
...    print(row.User, row.name)
<User(name='ed', fullname='Ed Jones', nickname='eddie')> ed
<User(name='wendy', fullname='Wendy Williams', nickname='windy')> wendy
<User(name='mary', fullname='Mary Contrary', nickname='mary')> mary
<User(name='fred', fullname='Fred Flintstone', nickname='freddy')> fred
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname,
        users.name AS users_name__1
FROM users
[...] ()
```

Вы можете управлять именами отдельных выражений столбцов с помощью конструкции [ColumnElement.label()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnElement.label), которая доступна из любого объекта, производного от [ColumnElement](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnElement), а также любого атрибута класса, который сопоставлен с ним (например, `User.name`):

```python
>>> for row in session.query(User.name.label('name_label')).all():
...    print(row.name_label)
ed
wendy
mary
fred
```

```sql
SELECT users.name AS name_label
FROM users
[...] ()
```

Имя, присвоенное полному объекту, такому как **User**, при условии, что в вызове [Session.query()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.query) присутствует несколько объектов, можно контролировать с помощью [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) :

```python
>>> from sqlalchemy.orm import aliased
>>> user_alias = aliased(User, name='user_alias')

>>> for row in session.query(user_alias, user_alias.name).all():
...    print(row.user_alias)
<User(name='ed', fullname='Ed Jones', nickname='eddie')>
<User(name='wendy', fullname='Wendy Williams', nickname='windy')>
<User(name='mary', fullname='Mary Contrary', nickname='mary')>
<User(name='fred', fullname='Fred Flintstone', nickname='freddy')>
```

```sql
SELECT user_alias.id AS user_alias_id,
        user_alias.name AS user_alias_name,
        user_alias.fullname AS user_alias_fullname,
        user_alias.nickname AS user_alias_nickname,
        user_alias.name AS user_alias_name__1
FROM users AS user_alias
[...] ()
```

Основные операции с [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) включают выдачу **LIMIT** и **OFFSET**, наиболее удобно использовать срезы массива Python и обычно в сочетании с **ORDER BY**:

```python
>>> for u in session.query(User).order_by(User.id)[1:3]:
...    print(u)
<User(name='wendy', fullname='Wendy Williams', nickname='windy')>
<User(name='mary', fullname='Mary Contrary', nickname='mary')>
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users ORDER BY users.id
LIMIT ? OFFSET ?
[...] (2, 1)
```

и фильтрация результатов, которая выполняется либо с помощью **filter\_by()**, использующей аргументы ключевого слова:

```python
>>> for name, in session.query(User.name).\
...             filter_by(fullname='Ed Jones'):
...    print(name)
ed
```

```sql
SELECT users.name AS users_name FROM users
WHERE users.fullname = ?
[...] ('Ed Jones',)
```

… или **filter()**, который использует более гибкие конструкции языка выражений SQL. Это позволяет вам использовать обычные операторы Python с атрибутами уровня класса в вашем сопоставленном классе:

```python
>>> for name, in session.query(User.name).\
...             filter(User.fullname=='Ed Jones'):
...    print(name)
ed
```

```sql
SELECT users.name AS users_name FROM users
WHERE users.fullname = ?
[...] ('Ed Jones',)
```

Объект [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) является полностью генеративным (**generative**), а это означает, что большинство вызовов методов возвращают новый объект **Query**, к которому можно добавить дополнительные критерии. Например, чтобы запросить пользователей с именем «ed» и полным именем «Ed Jones», вы можете дважды вызвать **filter()**, который объединяет критерии с помощью **AND**:

```python
>>> for user in session.query(User).\
...          filter(User.name=='ed').\
...          filter(User.fullname=='Ed Jones'):
...    print(user)
<User(name='ed', fullname='Ed Jones', nickname='eddie')>
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name = ? AND users.fullname = ?
[...] ('ed', 'Ed Jones')
```

### Общие операторы фильтра

Вот краткое изложение некоторых из наиболее распространенных операторов, используемых в **filter()**:

#### [ColumnOperators.\_\_eq\_\_()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.\_\_eq\_\_)

```python
query.filter(User.name == 'ed')
```

#### [ColumnOperators.\_\_ne\_\_()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.\_\_ne\_\_)

```python
query.filter(User.name != 'ed')
```

#### [ColumnOperators.like()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.like)

```python
query.filter(User.name.like('%ed%'))
```

{% hint style="info" %}
[ColumnOperators.like()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.like) отображает оператор **LIKE**, который нечувствителен к регистру в некоторых бэкендах и чувствителен к регистру в других. Для гарантированного сравнения без учета регистра используйте [ColumnOperators.ilike()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.ilike).
{% endhint %}

#### [ColumnOperators.ilike()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.ilike) (нечувствительный к регистру LIKE)

```python
query.filter(User.name.ilike('%ed%'))
```

{% hint style="info" %}
большинство бэкендов не поддерживают **ILIKE** напрямую. Для них оператор [ColumnOperators.ilike()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.ilike) отображает выражение, объединяющее **LIKE** с SQL-функцией **LOWER**, применяемой к каждому операнду.
{% endhint %}

#### [ColumnOperators.in\_()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.in\_)

```python
query.filter(User.name.in_(['ed', 'wendy', 'jack']))

# работает и с объектами запроса тоже:
query.filter(User.name.in_(
    session.query(User.name).filter(User.name.like('%ed%'))
))

# используйте tuple_() для составных (многостолбцовых) запросов
from sqlalchemy import tuple_
query.filter(
    tuple_(User.name, User.nickname).\
    in_([('ed', 'edsnickname'), ('wendy', 'windy')])
)
```

#### [ColumnOperators.not\_in()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.not\_in)

```python
query.filter(~User.name.in_(['ed', 'wendy', 'jack']))
```

#### [ColumnOperators.is\_()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.is\_)

```python
query.filter(User.name == None)

# в качестве альтернативы, если pep8/linters вызывают беспокойство
query.filter(User.name.is_(None))
```

#### [ColumnOperators.is\_not()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.is\_not)

```python
query.filter(User.name != None)

# в качестве альтернативы, если pep8/linters вызывают беспокойство
query.filter(User.name.is_not(None))
```

#### AND

```python
# используйте and_()
from sqlalchemy import and_
query.filter(and_(User.name == 'ed', User.fullname == 'Ed Jones'))

# или отправьте несколько выражений на .filter()
query.filter(User.name == 'ed', User.fullname == 'Ed Jones')

# или свяжите несколько вызовов filter()/filter_by()
query.filter(User.name == 'ed').filter(User.fullname == 'Ed Jones')
```

{% hint style="info" %}
Убедитесь, что вы используете [and\_()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.and\_), а не оператор Python **and**!
{% endhint %}

#### OR

```python
from sqlalchemy import or_
query.filter(or_(User.name == 'ed', User.name == 'wendy'))
```

{% hint style="info" %}
Убедитесь, что вы используете [or\_()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.or\_), а не Python оператор **or**!
{% endhint %}

#### [ColumnOperators.match()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.match)

```python
query.filter(User.name.match('wendy'))
```

{% hint style="info" %}
[ColumnOperators.match()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.match) использует специфичную для базы данных функцию **MATCH** или **CONTAINS**; его поведение зависит от бэкенда и недоступно для некоторых бэкендов, таких как SQLite.
{% endhint %}

### Возврат списков и скаляров

Ряд методов в [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) немедленно выдают SQL и возвращают значение, содержащее результаты загруженной базы данных. Вот краткий обзор:

#### [Query.all()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.all) возвращает список:

```python
>>> query = session.query(User).filter(User.name.like('%ed')).order_by(User.id)

>>> query.all()
[<User(name='ed', fullname='Ed Jones', nickname='eddie')>,
      <User(name='fred', fullname='Fred Flintstone', nickname='freddy')>]
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name LIKE ? ORDER BY users.id
[...] ('%ed',)
```

{% hint style="danger" %}
Когда объект [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) возвращает списки объектов с ORM-отображением, таких как объект **User** выше, записи дедублируются (**deduplicated**) на основе первичного ключа, поскольку результаты интерпретируются из набора результатов SQL. То есть, если SQL-запрос дважды возвращает строку с `id=7`, вы получите только один объект `User(id=7)` в списке результатов. Это не относится к случаю, когда запрашиваются отдельные столбцы.
{% endhint %}

{% hint style="info" %}
**Смотри также**

[Мой запрос не возвращает такое же количество объектов, как говорит мне query.count() - почему?](https://docs.sqlalchemy.org/en/14/faq/sessions.html#faq-query-deduplicating)
{% endhint %}

#### [Query.first()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.first) применяет ограничение, равное единице, и возвращает первый результат в виде скаляра:

```python
>>> query.first()
<User(name='ed', fullname='Ed Jones', nickname='eddie')>
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name LIKE ? ORDER BY users.id
 LIMIT ? OFFSET ?
[...] ('%ed', 1, 0)
```

#### [Query.one()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.one) полностью извлекает все строки, и если в результате присутствует не ровно один идентификатор объекта или составная строка, возникает ошибка. С несколькими найденными строками:

```python
>>> user = query.one()
Traceback (most recent call last):
...
MultipleResultsFound: Multiple rows were found for one()
```

Без найденных строк:

```python
>>> user = query.filter(User.id == 99).one()
Traceback (most recent call last):
...
NoResultFound: No row was found for one()
```

Метод [Query.one()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.one) отлично подходит для систем, которые предполагают по-разному обрабатывать **«элементы не найдены»** и **«найдено несколько элементов»**; например, веб-служба RESTful, которая может захотеть поднять **«404 not found»**, когда результаты не найдены, но вызвать ошибку приложения, когда найдено несколько результатов.

#### [Query.one\_or\_none()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.one\_or\_none) похож на [Query.one()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.one), за исключением того, что если результатов не найдено, он не вызывает ошибку; он просто возвращает `None`.

Однако, как и [Query.one()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.one), он вызывает ошибку, если найдено несколько результатов.

#### [Query.scalar()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.scalar) вызывает метод [Query.one()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.one) и в случае успеха возвращает первый столбец строки:

```python
>>> query = session.query(User.id).filter(User.name == 'ed').\
...    order_by(User.id)

>>> query.scalar()
1
```

```sql
SELECT users.id AS users_id
FROM users
WHERE users.name = ? ORDER BY users.id
[...] ('ed',)
```

### Использование текстового SQL

Литеральные строки можно гибко использовать с [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query), указав их использование с конструкцией [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text), которая принимается большинством применимых методов. Например, [Query.filter()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.filter) и [Query.order\_by()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.order\_by):

```python
>>> from sqlalchemy import text

>>> for user in session.query(User).\
...             filter(text("id<224")).\
...             order_by(text("id")).all():
...     print(user.name)
ed
wendy
mary
fred
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE id<224 ORDER BY id
[...] ()
```

Параметры привязки можно указать с помощью строкового SQL с использованием двоеточия. Чтобы указать значения, используйте метод [Query.params()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.params):

```python
>>> session.query(User).filter(text("id<:value and name=:name")).\
...     params(value=224, name='fred').order_by(User.id).one()
<User(name='fred', fullname='Fred Flintstone', nickname='freddy')>
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE id<? and name=? ORDER BY users.id
[...] (224, 'fred')
```

Чтобы использовать оператор, полностью основанный на строке, конструкцию [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text), представляющую полный оператор, можно передать в [Query.from\_statement()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.from\_statement). Без дополнительной спецификации ORM будет сопоставлять столбцы в сопоставлении ORM с результатом, возвращаемым оператором SQL, на основе имени столбца:

```python
>>> session.query(User).from_statement(
...  text("SELECT * FROM users where name=:name")).params(name='ed').all()
[<User(name='ed', fullname='Ed Jones', nickname='eddie')>]
```

```sql
SELECT * FROM users where name=?
[...] ('ed',)
```

Для лучшего нацеливания сопоставленных столбцов на текстовый **SELECT**, а также для сопоставления определенного подмножества столбцов в произвольном порядке отдельные сопоставленные столбцы передаются в нужном порядке в [TextClause.columns()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.TextClause.columns):

```python
>>> stmt = text("SELECT name, id, fullname, nickname "
...             "FROM users where name=:name")
>>> stmt = stmt.columns(User.name, User.id, User.fullname, User.nickname)

>>> session.query(User).from_statement(stmt).params(name='ed').all()
[<User(name='ed', fullname='Ed Jones', nickname='eddie')>]
```

```sql
SELECT name, id, fullname, nickname FROM users where name=?
[...] ('ed',)
```

При выборе из конструкции [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text) запрос может по-прежнему указывать, какие столбцы и объекты должны быть возвращены; вместо `query(User)` мы также можем запросить столбцы по отдельности, как и в любом другом случае:

```python
>>> stmt = text("SELECT name, id FROM users where name=:name")
>>> stmt = stmt.columns(User.name, User.id)

>>> session.query(User.id, User.name).\
...          from_statement(stmt).params(name='ed').all()
[(1, u'ed')]
```

```sql
SELECT name, id FROM users where name=?
[...] ('ed',)
```

{% hint style="info" %}
Смотри также

[Использование текстового SQL](https://docs.sqlalchemy.org/en/14/core/tutorial.html#sqlexpression-text) — конструкция [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text) объяснена с точки зрения запросов только для ядра.
{% endhint %}

### Подсчет (Counting)

[Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) включает удобный метод подсчета, который называется [Query.count()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.count):

```python
>>> session.query(User).filter(User.name.like('%ed')).count()
2
```

```sql
SELECT count(*) AS count_1
FROM (SELECT users.id AS users_id,
                users.name AS users_name,
                users.fullname AS users_fullname,
                users.nickname AS users_nickname
FROM users
WHERE users.name LIKE ?) AS anon_1
[...] ('%ed',)
```

{% hint style="info" %}
**Подсчет count()**

[Query.count()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.count) раньше был очень сложным методом, когда он пытался угадать, нужен ли подзапрос вокруг существующего запроса, и в некоторых экзотических случаях он не работал правильно. Теперь, когда он каждый раз использует простой подзапрос, он занимает всего две строки и всегда возвращает правильный ответ. Используйте **func.count()**, если конкретный оператор абсолютно не может допустить присутствия подзапроса.
{% endhint %}

Метод [Query.count()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.count) используется для определения того, сколько строк будет возвращено оператором SQL. Глядя на сгенерированный SQL выше, SQLAlchemy всегда помещает все, что мы запрашиваем, в подзапрос, а затем подсчитывает строки из него. В некоторых случаях это может быть сведено к более простому `SELECT count(*) FROM table`, однако современные версии SQLAlchemy не пытаются угадать, когда это уместно, поскольку точный SQL может быть сгенерирован с использованием более явных средств.

В ситуациях, когда необходимо конкретно указать **«вещь, подлежащую подсчету»**, мы можем указать функцию **«count»** напрямую, используя выражение **func.count()**, доступное из конструкции [expression.func](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.func). Ниже мы используем его для возврата количества каждого отдельного имени пользователя:

```python
>>> from sqlalchemy import func

>>> session.query(func.count(User.name), User.name).group_by(User.name).all()
[(1, u'ed'), (1, u'fred'), (1, u'mary'), (1, u'wendy')]
```

```sql
SELECT count(users.name) AS count_1, users.name AS users_name
FROM users GROUP BY users.name
[...] ()
```

Чтобы получить наш простой `SELECT count(*) FROM table`, мы можем применить ее как:

```python
>>> session.query(func.count('*')).select_from(User).scalar()
4
```

```sql
SELECT count(?) AS count_1
FROM users
[...] ('*',)
```

Использование [Query.select\_from()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.select\_from) может быть удалено, если мы напрямую выразим количество с точки зрения первичного ключа пользователя **User**:

```python
>>> session.query(func.count(User.id)).scalar()
4
```

```sql
SELECT count(users.id) AS count_1
FROM users
[...] ()
```
