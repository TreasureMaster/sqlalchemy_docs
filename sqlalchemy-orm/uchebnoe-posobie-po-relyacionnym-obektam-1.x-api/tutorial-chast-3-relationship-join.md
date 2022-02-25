# Tutorial часть 3 (relationship, join)

## Построение связей (relationship)

Давайте рассмотрим, как можно отображать и запрашивать вторую таблицу, связанную с **User**. Пользователи в нашей системе могут хранить любое количество адресов электронной почты, связанных с их именем пользователя. Это подразумевает базовую ассоциацию пользователей _**users**_ «один ко многим» с новой таблицей, в которой хранятся адреса электронной почты, которую мы будем называть _**addresses**_. Используя декларативный подход, мы определяем эту таблицу вместе с ее сопоставленным классом **Address**:

```python
>>> from sqlalchemy import ForeignKey
>>> from sqlalchemy.orm import relationship

>>> class Address(Base):
...     __tablename__ = 'addresses'
...     id = Column(Integer, primary_key=True)
...     email_address = Column(String, nullable=False)
...     user_id = Column(Integer, ForeignKey('users.id'))
...
...     user = relationship("User", back_populates="addresses")
...
...     def __repr__(self):
...         return "<Address(email_address='%s')>" % self.email_address

>>> User.addresses = relationship(
...     "Address", order_by=Address.id, back_populates="user")
```

В приведенном выше классе представлена ​​конструкция [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey), которая представляет собой директиву, применяемую к столбцу, указывающую, что значения в этом столбце должны быть ограничены значениями, присутствующими в именованном удаленном столбце. Это ключевая особенность реляционных баз данных и «клей», который преобразует несвязанный набор таблиц, чтобы иметь богатые перекрывающиеся отношения. Вышеупомянутый [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey) указывает, что значения в столбце `address.user_id` должны быть ограничены значениями в столбце `users.id`, т. е. его первичным ключом.

Вторая директива, известная как [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), сообщает ORM, что сам класс **Address** должен быть связан с классом **User** с помощью атрибута `Address.user`. [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) использует отношения внешнего ключа между двумя таблицами, чтобы определить характер этой связи, определяя, что `Address.user` будет много к одному ([many to one](https://docs.sqlalchemy.org/en/14/glossary.html#term-many-to-one)). Дополнительная директива [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) помещается в сопоставленный класс **User** под атрибутом `User.addresses`. В обеих директивах [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) параметр [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) назначается для ссылки на дополнительные имена атрибутов; таким образом каждое [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) может принять разумное решение относительно той же связи, выраженной в обратном порядке; с одной стороны, `Address.user` относится к экземпляру **User**, а с другой стороны, `User.addresses` относится к списку экземпляров **Address**.

{% hint style="info" %}
Параметр [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) — это более новая версия очень распространенной функции SQLAlchemy, которая называется [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref). Параметр `relationship.backref` никуда не делся и всегда будет оставаться доступным! [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) — это то же самое, за исключением того, что оно немного более подробное и им легче манипулировать. Обзор всей темы см. в разделе «[Связывание отношений с обратной ссылкой](https://docs.sqlalchemy.org/en/14/orm/backref.html#relationships-backref)».
{% endhint %}

Обратная сторона отношения «многие к одному» — всегда «один ко многим» ([one to many](https://docs.sqlalchemy.org/en/14/glossary.html#term-one-to-many)). Полный каталог доступных конфигураций [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) находится в Базовых шаблонах отношений ([Basic Relationship Patterns](https://docs.sqlalchemy.org/en/14/orm/basic\_relationships.html#relationship-patterns)).

Два взаимодополняющих отношения `Address.user` и `User.addresses` называются двунаправленными отношениями и являются ключевой функцией ORM SQLAlchemy. В разделе «Связывание отношений с обратной ссылкой» ([Linking Relationships with Backref](https://docs.sqlalchemy.org/en/14/orm/backref.html#relationships-backref)) подробно обсуждается функция «обратной ссылки».

Аргументы для [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), которые относятся к удаленному классу, могут быть указаны с использованием строк, при условии, что используется декларативная система. После завершения всех сопоставлений эти строки оцениваются как выражения Python, чтобы создать фактический аргумент, в приведенном выше случае класс **User**. Имена, которые разрешены во время этой оценки, включают, среди прочего, имена всех классов, которые были созданы с точки зрения объявленной базы.

См. строку документации для [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) для более подробной информации о стиле аргумента.

{% hint style="info" %}
**Вы знали ?**

* ограничение **FOREIGN KEY** в большинстве (хотя и не во всех) реляционных базах данных может ссылаться только на столбец первичного ключа или столбец с ограничением **UNIQUE**.
* ограничение **FOREIGN KEY**, которое ссылается на первичный ключ с несколькими столбцами и само имеет несколько столбцов, известно как «составной внешний ключ». Он также может ссылаться на подмножество этих столбцов.
* Столбцы **FOREIGN KEY** могут автоматически обновляться в ответ на изменение в указанном столбце или строке. Это известно как _ссылочное действие_ **CASCADE** и является встроенной функцией реляционной базы данных.
* **FOREIGN KEY** может ссылаться на свою собственную таблицу. Это называется «самореферентным» внешним ключом.
* Узнайте больше о внешних ключах в разделе [Внешний ключ — Википедия](https://en.wikipedia.org/wiki/Foreign\_key).
{% endhint %}

Нам нужно будет создать таблицу _**addresses**_ в базе данных, поэтому мы выполним еще один **CREATE** из наших метаданных, который **пропустит уже созданные таблицы**:

```python
>>> Base.metadata.create_all(engine)
```

```sql
BEGIN...
CREATE TABLE addresses (
    id INTEGER NOT NULL,
    email_address VARCHAR NOT NULL,
    user_id INTEGER,
    PRIMARY KEY (id),
    FOREIGN KEY(user_id) REFERENCES users (id)
)
[...] ()
COMMIT
```

## Работа со связанными объектами

Теперь, когда мы создаем пользователя **User**, будет присутствовать пустая коллекция адресов _**addresses**_. Здесь возможны различные типы коллекций, такие как множества set и словари dict (подробности см. в разделе «Настройка доступа к коллекциям» - [Customizing Collection Access](https://docs.sqlalchemy.org/en/14/orm/collections.html#custom-collections)), но по умолчанию коллекция представляет собой список Python list.

```python
>>> jack = User(name='jack', fullname='Jack Bean', nickname='gjffdd')
>>> jack.addresses
[]
```

Мы можем добавлять объекты **Address** в наш объект **User**. В этом случае мы просто присваиваем полный список напрямую:

```python
>>> jack.addresses = [
...                 Address(email_address='jack@google.com'),
...                 Address(email_address='j25@yahoo.com')]
```

При использовании двунаправленной связи элементы, добавленные в одном направлении, автоматически становятся видимыми в другом направлении. Это поведение происходит на основе событий изменения атрибута и оценивается в Python без использования SQL:

```python
>>> jack.addresses[1]
<Address(email_address='j25@yahoo.com')>

>>> jack.addresses[1].user
<User(name='jack', fullname='Jack Bean', nickname='gjffdd')>
```

Давайте добавим и зафиксируем _**Jack Bean**_ в базе данных. _**jack**_, а также два члена **Address** в соответствующей коллекции адресов добавляются в сеанс одновременно с использованием процесса, известного как каскадирование (**cascading**):

```python
>>> session.add(jack)
>>> session.commit()
```

```sql
INSERT INTO users (name, fullname, nickname) VALUES (?, ?, ?)
[...] ('jack', 'Jack Bean', 'gjffdd')
INSERT INTO addresses (email_address, user_id) VALUES (?, ?)
[...] ('jack@google.com', 5)
INSERT INTO addresses (email_address, user_id) VALUES (?, ?)
[...] ('j25@yahoo.com', 5)
COMMIT
```

Запрашивая Джека, мы получаем только Джека. Для адресов Джека SQL еще не выдан:

```python
>>> jack = session.query(User).\
... filter_by(name='jack').one()
>>> jack
<User(name='jack', fullname='Jack Bean', nickname='gjffdd')>
```

```sql
BEGIN (implicit)
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name = ?
[...] ('jack',)
```

Давайте посмотрим на коллекцию адресов _**addresses**_. Смотрите SQL:

```python
>>> jack.addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]
```

```sql
SELECT addresses.id AS addresses_id,
        addresses.email_address AS
        addresses_email_address,
        addresses.user_id AS addresses_user_id
FROM addresses
WHERE ? = addresses.user_id ORDER BY addresses.id
[...] (5,)
```

Когда мы обращались к коллекции _**addresses**_, внезапно выдавался SQL. Это пример отношения ленивой загрузки ([lazy loading](https://docs.sqlalchemy.org/en/14/glossary.html#term-lazy-loading)). Коллекция адресов загружена и ведет себя как обычный список. Чуть позже мы рассмотрим способы оптимизации загрузки этой коллекции.

## Запросы с соединениями (Quering with Joins)

Теперь, когда у нас есть две таблицы, мы можем показать некоторые дополнительные возможности [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query), в частности, как создавать запросы, которые работают с обеими таблицами одновременно. [Страница Википедии, посвященная SQL JOIN](https://en.wikipedia.org/wiki/Join\_\(SQL\)), предлагает хорошее введение в методы соединения, некоторые из которых мы проиллюстрируем здесь.

Чтобы построить простое неявное соединение между пользователем **User** и адресом **Address**, мы можем использовать [Query.filter()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.filter), чтобы соединить их связанные столбцы вместе. Ниже мы загружаем объекты **User** и **Address** сразу, используя этот метод:

```python
>>> for u, a in session.query(User, Address).\
...                     filter(User.id==Address.user_id).\
...                     filter(Address.email_address=='jack@google.com').\
...                     all():
...     print(u)
...     print(a)
<User(name='jack', fullname='Jack Bean', nickname='gjffdd')>
<Address(email_address='jack@google.com')>
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname,
        addresses.id AS addresses_id,
        addresses.email_address AS addresses_email_address,
        addresses.user_id AS addresses_user_id
FROM users, addresses
WHERE users.id = addresses.user_id
        AND addresses.email_address = ?
[...] ('jack@google.com',)
```

С другой стороны, реальный синтаксис SQL **JOIN** проще всего реализовать с помощью метода [Query.join()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.join):

```python
>>> session.query(User).join(Address).\
...         filter(Address.email_address=='jack@google.com').\
...         all()
[<User(name='jack', fullname='Jack Bean', nickname='gjffdd')>]
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users JOIN addresses ON users.id = addresses.user_id
WHERE addresses.email_address = ?
[...] ('jack@google.com',)
```

[Query.join()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.join) знает, как соединить пользователя **User** и адрес **Address**, потому что между ними есть только один внешний ключ. Если внешних ключей не было или их было несколько, [Query.join()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.join) работает лучше, когда используется одна из следующих форм:

```python
query.join(Address, User.id==Address.user_id)          # явное условие
query.join(User.addresses)                             # указать отношение слева направо
query.join(Address, User.addresses)                    # то же, с явной целью
query.join(User.addresses.and_(Address.name != 'foo')) # использовать отношения + дополнительные критерии ON
```

Как и следовало ожидать, та же идея используется для «внешних» соединений с использованием функции [Query.outerjoin()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.outerjoin):

```python
query.outerjoin(User.addresses)   # LEFT OUTER JOIN
```

Справочная документация по [Query.join()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.join) содержит подробную информацию и примеры стилей вызова, принимаемых этим методом; `Query.join()` — это важный метод, который используется в любом приложении, свободно работающем с SQL.

{% hint style="info" %}
**Что выбирает Query, если существует несколько сущностей?**

Метод [Query.join()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.join) обычно выполняет **соединение из крайнего левого элемента в списке сущностей**, когда предложение **ON** опущено или если предложение **ON** является простым выражением SQL. Чтобы управлять первым объектом в списке **JOIN**, используйте метод [Query.select\_from()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.select\_from):

```python
query = session.query(User, Address).select_from(Address).join(User)
```
{% endhint %}

### Использование псевдонимов (aliases)

При выполнении запросов по нескольким таблицам, если на одну и ту же таблицу нужно ссылаться более одного раза, SQL обычно требует, чтобы эта таблица имела псевдоним с другим именем, чтобы ее можно было отличить от других вхождений этой таблицы. Это поддерживается с помощью конструкции [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased). При присоединении к отношениям с использованием [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) можно использовать специальный метод атрибута [PropComparator.of\_type()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator.of\_type), чтобы изменить цель соединения отношений, чтобы она ссылалась на данный объект `aliased()`. Ниже мы дважды присоединяемся к объекту **Address**, чтобы найти пользователя, у которого одновременно есть два разных адреса электронной почты:

```python
>>> from sqlalchemy.orm import aliased
>>> adalias1 = aliased(Address)
>>> adalias2 = aliased(Address)

>>> for username, email1, email2 in \
...     session.query(User.name, adalias1.email_address, adalias2.email_address).\
...     join(User.addresses.of_type(adalias1)).\
...     join(User.addresses.of_type(adalias2)).\
...     filter(adalias1.email_address=='jack@google.com').\
...     filter(adalias2.email_address=='j25@yahoo.com'):
...     print(username, email1, email2)
jack jack@google.com j25@yahoo.com
```

```sql
SELECT users.name AS users_name,
        addresses_1.email_address AS addresses_1_email_address,
        addresses_2.email_address AS addresses_2_email_address
FROM users JOIN addresses AS addresses_1
        ON users.id = addresses_1.user_id
JOIN addresses AS addresses_2
        ON users.id = addresses_2.user_id
WHERE addresses_1.email_address = ?
        AND addresses_2.email_address = ?
[...] ('jack@google.com', 'j25@yahoo.com')
```

В дополнение к использованию метода [PropComparator.of\_type()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator.of\_type) часто можно увидеть, как метод [Query.join()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.join) присоединяется к определенной цели, указав ее отдельно:

```python
# эквивалентно query.join(User.addresses.of_type(adalias1))
q = query.join(adalias1, User.addresses)
```

### Использование подзапросов

[Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) подходит для создания операторов, которые можно использовать в качестве подзапросов. Предположим, мы хотим загрузить объекты **User** вместе с количеством записей **Address**, которые есть у каждого пользователя. Лучший способ сгенерировать SQL, подобный этому, — получить количество адресов, сгруппированных по идентификаторам пользователей, и **JOIN** к родителю. В этом случае мы используем **LEFT OUTER JOIN**, чтобы получить строки для тех пользователей, у которых нет адресов, например:

```sql
SELECT users.*, adr_count.address_count FROM users LEFT OUTER JOIN
    (SELECT user_id, count(*) AS address_count
        FROM addresses GROUP BY user_id) AS adr_count
    ON users.id=adr_count.user_id
```

Используя [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query), мы строим подобный оператор изнутри. Средство доступа _**statement**_ к операторам возвращает SQL-выражение, представляющее оператор, сгенерированный конкретным запросом `Query`, — это экземпляр конструкции [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), которая описана в учебнике по языку выражений SQL ([SQL Expression Language Tutorial (1.x API)](https://docs.sqlalchemy.org/en/14/core/tutorial.html)):

```python
>>> from sqlalchemy.sql import func
>>> stmt = session.query(Address.user_id, func.count('*').\
...         label('address_count')).\
...         group_by(Address.user_id).subquery()
```

Ключевое слово **func** генерирует функции SQL, а метод **subquery()** в [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) создает конструкцию выражения SQL, представляющую оператор **SELECT**, встроенный в псевдоним (на самом деле это сокращение для `query.statement.alias()`).

Когда у нас есть оператор, он ведет себя как конструктор таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), такой как та, которую мы создали для пользователей _**users**_ в начале этого руководства. Столбцы оператора доступны через атрибут с именем _**c**_:

```python
>>> for u, count in session.query(User, stmt.c.address_count).\
...     outerjoin(stmt, User.id==stmt.c.user_id).order_by(User.id):
...     print(u, count)
<User(name='ed', fullname='Ed Jones', nickname='eddie')> None
<User(name='wendy', fullname='Wendy Williams', nickname='windy')> None
<User(name='mary', fullname='Mary Contrary', nickname='mary')> None
<User(name='fred', fullname='Fred Flintstone', nickname='freddy')> None
<User(name='jack', fullname='Jack Bean', nickname='gjffdd')> 2
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname,
        anon_1.address_count AS anon_1_address_count
FROM users LEFT OUTER JOIN
    (SELECT addresses.user_id AS user_id, count(?) AS address_count
    FROM addresses GROUP BY addresses.user_id) AS anon_1
    ON users.id = anon_1.user_id
ORDER BY users.id
[...] ('*',)
```

### Выбор сущностей из подзапросов

Выше мы только что выбрали результат, который включал столбец из подзапроса. Что, если бы мы хотели, чтобы наш подзапрос отображался на сущность? Для этого мы используем `aliased()`, чтобы связать «псевдоним» сопоставленного класса с подзапросом:

```python
>>> stmt = session.query(Address).\
...                 filter(Address.email_address != 'j25@yahoo.com').\
...                 subquery()
>>> addr_alias = aliased(Address, stmt)
>>> for user, address in session.query(User, addr_alias).\
...         join(addr_alias, User.addresses):
...     print(user)
...     print(address)
<User(name='jack', fullname='Jack Bean', nickname='gjffdd')>
<Address(email_address='jack@google.com')>
```

```sql
SELECT users.id AS users_id,
            users.name AS users_name,
            users.fullname AS users_fullname,
            users.nickname AS users_nickname,
            anon_1.id AS anon_1_id,
            anon_1.email_address AS anon_1_email_address,
            anon_1.user_id AS anon_1_user_id
FROM users JOIN
    (SELECT addresses.id AS id,
            addresses.email_address AS email_address,
            addresses.user_id AS user_id
    FROM addresses
    WHERE addresses.email_address != ?) AS anon_1
    ON users.id = anon_1.user_id
[...] ('j25@yahoo.com',)
```

### Использование EXISTS

Ключевое слово **EXISTS** в SQL — это логический оператор, который возвращает `True`, если данное выражение содержит какие-либо строки. Он может использоваться во многих сценариях вместо объединений, а также полезен для поиска строк, которым нет соответствующей строки в связанной таблице.

Существует явная конструкция **EXISTS**, которая выглядит так:

```python
>>> from sqlalchemy.sql import exists
>>> stmt = exists().where(Address.user_id==User.id)

>>> for name, in session.query(User.name).filter(stmt):
...     print(name)
jack
```

```sql
SELECT users.name AS users_name
FROM users
WHERE EXISTS (SELECT *
FROM addresses
WHERE addresses.user_id = users.id)
[...] ()
```

В запросе [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) есть несколько операторов, которые автоматически используют **EXISTS**. Выше оператор может быть выражен для отношения `User.addresses` с помощью [Comparator.any()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator.any):

```python
>>> for name, in session.query(User.name).\
...         filter(User.addresses.any()):
...     print(name)
jack
```

```sql
SELECT users.name AS users_name
FROM users
WHERE EXISTS (SELECT 1
FROM addresses
WHERE users.id = addresses.user_id)
[...] ()
```

[Comparator.any()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator.any) также принимает критерий, чтобы ограничить совпадающие строки:

```python
>>> for name, in session.query(User.name).\
...     filter(User.addresses.any(Address.email_address.like('%google%'))):
...     print(name)
jack
```

```sql
SELECT users.name AS users_name
FROM users
WHERE EXISTS (SELECT 1
FROM addresses
WHERE users.id = addresses.user_id AND addresses.email_address LIKE ?)
[...] ('%google%',)
```

[Comparator.has()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator.has) — это тот же оператор, что и [Comparator.any()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator.any) для отношений «многие к одному» (здесь также обратите внимание на оператор `~`, который означает **«NOT»**):

```python
>>> session.query(Address).\
...         filter(~Address.user.has(User.name=='jack')).all()
[]
```

```sql
SELECT addresses.id AS addresses_id,
        addresses.email_address AS addresses_email_address,
        addresses.user_id AS addresses_user_id
FROM addresses
WHERE NOT (EXISTS (SELECT 1
FROM users
WHERE users.id = addresses.user_id AND users.name = ?))
[...] ('jack',)
```

### Общие операторы relationship связей

Вот все операторы, которые строятся на отношениях **relationship** — каждый из них связан со своей документацией по API, которая включает полную информацию об использовании и поведении:

#### [Comparator.\_\_eq\_\_()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator.\_\_eq\_\_) (сравнение "равно" для связи "многие к одному")

```python
query.filter(Address.user == someuser)
```

#### [Comparator.\_\_ne\_\_()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator.\_\_ne\_\_) (сравнение "не равно" для связи "многие к одному")

```python
query.filter(Address.user != someuser)
```

#### IS NULL (сравнение в связи "многие к одному", также использует [Comaprator.\_\_eq\_\_()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator.\_\_eq\_\_))

```python
query.filter(Address.user == None)
```

#### [Comparator.contains()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator.contains) (используется для коллекций и связи "один ко многим")

```python
query.filter(User.addresses.contains(someaddress))
```

#### [Comparator.any()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator.any) (используется для коллекций)

```python
query.filter(User.addresses.any(Address.email_address == 'bar'))

# также принимает аргументы ключевого слова:
query.filter(User.addresses.any(email_address='bar'))
```

#### [Comparator.has()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator.has) (используется для скалярных ссылок)

```python
query.filter(Address.user.has(name='ed'))
```

#### [Query.with\_parent()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.with\_parent) (используется для любых связей)

```python
session.query(Address).with_parent(someuser, 'addresses')
```
