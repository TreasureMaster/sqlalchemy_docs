# Tutorial часть 4 (eager load, delete)

## Нетерпеливая загрузка (eager loading)

Вспомните ранее, что мы проиллюстрировали операцию ленивой загрузки ([lazy loading](https://docs.sqlalchemy.org/en/14/glossary.html#term-lazy-loading)), когда мы обращались к коллекции `User.addresses` пользователя **User** и выдавался SQL. Если вы хотите сократить количество запросов (во многих случаях резко), мы можем применить нетерпеливую нагрузку ([eager load](https://docs.sqlalchemy.org/en/14/glossary.html#term-eager-load)) к операции запроса. SQLAlchemy предлагает три типа быстрой загрузки, два из которых являются автоматическими, а третий использует настраиваемый критерий. Все три обычно вызываются с помощью функций, известных как параметры запроса, которые дают дополнительные инструкции для запроса [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) о том, как мы хотели бы загружать различные атрибуты, с помощью метода [Query.options()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.options).

### Загрузка selectin

В этом случае мы хотели бы указать, что `User.addresses` должны загружаться с нетерпением. Хорошим выбором для загрузки набора объектов, а также связанных с ними коллекций является опция [selectinload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.selectinload), которая выдает второй оператор **SELECT**, который полностью загружает коллекции, связанные с только что загруженными результатами. Название **«selectin»** происходит от того факта, что оператор **SELECT** использует предложение **IN** для одновременного поиска связанных строк для нескольких объектов:

```python
>>> from sqlalchemy.orm import selectinload

>>> jack = session.query(User).\
...                 options(selectinload(User.addresses)).\
...                 filter_by(name='jack').one()
>>> jack
<User(name='jack', fullname='Jack Bean', nickname='gjffdd')>

>>> jack.addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name = ?
[...] ('jack',)
SELECT addresses.user_id AS addresses_user_id,
        addresses.id AS addresses_id,
        addresses.email_address AS addresses_email_address
FROM addresses
WHERE addresses.user_id IN (?)
ORDER BY addresses.id
[...] (5,)
```

### Загрузка joined

Другая функция автоматической нетерпеливой загрузки более известна и называется [joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload). Этот стиль загрузки создает **JOIN**, по умолчанию **LEFT OUTER JOIN**, так что ведущий объект, а также связанный объект или коллекция загружаются за один шаг. Мы иллюстрируем загрузку одной и той же коллекции _**addresses**_ таким образом — обратите внимание, что даже несмотря на то, что коллекция `User.addresses` на jack фактически заполнена прямо сейчас, запрос выдаст дополнительное соединение независимо от того, что:

```python
>>> from sqlalchemy.orm import joinedload

>>> jack = session.query(User).\
...                        options(joinedload(User.addresses)).\
...                        filter_by(name='jack').one()
>>> jack
<User(name='jack', fullname='Jack Bean', nickname='gjffdd')>

>>> jack.addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname,
        addresses_1.id AS addresses_1_id,
        addresses_1.email_address AS addresses_1_email_address,
        addresses_1.user_id AS addresses_1_user_id
FROM users
    LEFT OUTER JOIN addresses AS addresses_1 ON users.id = addresses_1.user_id
WHERE users.name = ? ORDER BY addresses_1.id
[...] ('jack',)
```

Обратите внимание, что несмотря на то, что **OUTER JOIN** привел к двум строкам, мы все равно получили только один экземпляр **User**. Это связано с тем, что [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) применяет стратегию «уникальности» (uniquing), основанную на идентификаторе объекта, к возвращаемым объектам. Это сделано специально для того, чтобы можно было применить объединенную нетерпеливую загрузку, не влияя на результаты запроса.

Хотя [joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload) существует уже давно, [selectinload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.selectinload) — это более новая форма быстрой загрузки. [selectinload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.selectinload) больше подходит для загрузки связанных коллекций, в то время как [joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload) лучше подходит для отношений «многие к одному» из-за того, что загружается только одна строка как для интереса, так и для связанного объекта. Также существует другая форма загрузки, [subqueryload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.subqueryload), которую можно использовать вместо [selectinload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.selectinload) при использовании составных первичных ключей на определенных бэкендах.

{% hint style="info" %}
**`joinedload()` не является заменой `join()`**

Соединение, созданное методом [joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload), имеет анонимный псевдоним, поэтому он **не влияет на результаты запроса**. Вызовы [Query.order\_by()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.order\_by) или [Query.filter()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.filter) **не могут** ссылаться на эти таблицы с псевдонимами — так называемые соединения «пространства пользователя» создаются с использованием [Query.join()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.join). Обоснование этого заключается в том, что [joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload) применяется только для того, чтобы влиять на то, как связанные объекты или коллекции загружаются в качестве детали оптимизации — их можно добавлять или удалять без влияния на фактические результаты. Подробное описание того, как это используется, см. в разделе «Дзен объединенной жадной загрузки» ([The Zen of Joined Eager Loading](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#zen-of-eager-loading)).
{% endhint %}

### Явное объединение + eagerload

Третий стиль нетерпеливой загрузки — это когда мы явно создаем **JOIN**, чтобы найти основные строки, и хотели бы дополнительно применить дополнительную таблицу к связанному объекту или коллекции основного объекта. Эта функция предоставляется через функцию [contains\_eager()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.contains\_eager) и обычно полезна для предварительной загрузки объекта «многие к одному» в запросе, который необходимо отфильтровать по тому же объекту. Ниже мы иллюстрируем загрузку строки **Address**, а также связанного с ней объекта **User**, фильтрацию по пользователю **User** с именем «jack» и использование [contains\_eager()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.contains\_eager) для применения столбцов «user» к атрибуту `Address.user`:

```python
>>> from sqlalchemy.orm import contains_eager

>>> jacks_addresses = session.query(Address).\
...                             join(Address.user).\
...                             filter(User.name=='jack').\
...                             options(contains_eager(Address.user)).\
...                             all()
>>> jacks_addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]

>>> jacks_addresses[0].user
<User(name='jack', fullname='Jack Bean', nickname='gjffdd')>
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname,
        addresses.id AS addresses_id,
        addresses.email_address AS addresses_email_address,
        addresses.user_id AS addresses_user_id
FROM addresses JOIN users ON users.id = addresses.user_id
WHERE users.name = ?
[...] ('jack',)
```

Для получения дополнительной информации о нетерпеливой загрузке, в том числе о том, как настроить различные формы загрузки по умолчанию, см. раздел Методы загрузки отношений ([Relationship Loading Techniques](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html)).

## Удаление

Давайте попробуем удалить _**jack**_ и посмотрим, что из этого выйдет. Мы отметим объект как удаленный в сеансе, затем выполним запрос _**count**_, чтобы убедиться, что строк не осталось:

```python
>>> session.delete(jack)

>>> session.query(User).filter_by(name='jack').count()
0
```

```sql
UPDATE addresses SET user_id=? WHERE addresses.id = ?
[...] ((None, 1), (None, 2))
DELETE FROM users WHERE users.id = ?
[...] (5,)
SELECT count(*) AS count_1
FROM (SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name = ?) AS anon_1
[...] ('jack',)
```

Все идет нормально. Как насчет объектов **Address** Джека?

```python
>>> session.query(Address).filter(
...     Address.email_address.in_(['jack@google.com', 'j25@yahoo.com'])
...  ).count()
2
```

```sql
SELECT count(*) AS count_1
FROM (SELECT addresses.id AS addresses_id,
                addresses.email_address AS addresses_email_address,
                addresses.user_id AS addresses_user_id
FROM addresses
WHERE addresses.email_address IN (?, ?)) AS anon_1
[...] ('jack@google.com', 'j25@yahoo.com')
```

Угу, они еще есть! Анализируя чистый SQL, мы видим, что столбец _**user\_id**_ каждого адреса был установлен в **NULL**, но строки не были удалены. SQLAlchemy не предполагает, что удаление каскадное, вы должны сообщить ему об этом.

### Настройка каскадного удаления/удаления-зомби

Мы настроим **каскадные** параметры отношения `User.addresses`, чтобы изменить поведение. Хотя SQLAlchemy позволяет добавлять новые атрибуты и отношения к сопоставлениям в любой момент времени, в этом случае существующее отношение необходимо удалить, поэтому нам нужно полностью удалить сопоставления и начать заново — мы закроем сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session):

```python
>>> session.close()
ROLLBACK
```

и используйте новый [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base):

```python
>>> Base = declarative_base()
```

Затем мы объявим класс **User**, добавив отношение _**addresses**_, включая конфигурацию каскада (конструктор мы тоже опустим):

```python
>>> class User(Base):
...     __tablename__ = 'users'
...
...     id = Column(Integer, primary_key=True)
...     name = Column(String)
...     fullname = Column(String)
...     nickname = Column(String)
...
...     addresses = relationship("Address", back_populates='user',
...                     cascade="all, delete, delete-orphan")
...
...     def __repr__(self):
...        return "<User(name='%s', fullname='%s', nickname='%s')>" % (
...                                self.name, self.fullname, self.nickname)
```

Затем мы воссоздаем **Address**, отметив, что в этом случае мы уже создали отношение `Address.user` через класс **User**:

```python
>>> class Address(Base):
...     __tablename__ = 'addresses'
...     id = Column(Integer, primary_key=True)
...     email_address = Column(String, nullable=False)
...     user_id = Column(Integer, ForeignKey('users.id'))
...     user = relationship("User", back_populates="addresses")
...
...     def __repr__(self):
...         return "<Address(email_address='%s')>" % self.email_address
```

Теперь, когда мы загружаем jack (ниже с помощью [Query.get()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.get), который загружается по первичному ключу), удаление адреса из соответствующей коллекции _**addresses**_ приведет к удалению этого **Address**:

```python
# загрузить Jack по первичному ключу
>>> jack = session.get(User, 5)
```

```sql
BEGIN (implicit)
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.id = ?
[...] (5,)
```

```python
# удаление одного Address (ленивая загрузка срабатывает)
>>> del jack.addresses[1]
```

```sql
SELECT addresses.id AS addresses_id,
        addresses.email_address AS addresses_email_address,
        addresses.user_id AS addresses_user_id
FROM addresses
WHERE ? = addresses.user_id
[...] (5,)
```

```python
# остался только один адрес
>>> session.query(Address).filter(
...     Address.email_address.in_(['jack@google.com', 'j25@yahoo.com'])
... ).count()
1
```

```sql
DELETE FROM addresses WHERE addresses.id = ?
[...] (2,)
SELECT count(*) AS count_1
FROM (SELECT addresses.id AS addresses_id,
                addresses.email_address AS addresses_email_address,
                addresses.user_id AS addresses_user_id
FROM addresses
WHERE addresses.email_address IN (?, ?)) AS anon_1
[...] ('jack@google.com', 'j25@yahoo.com')
```

Удаление Джека удалит и Джека, и оставшийся адрес **Address**, связанный с пользователем:

```python
>>> session.delete(jack)

>>> session.query(User).filter_by(name='jack').count()
0
```

```sql
DELETE FROM addresses WHERE addresses.id = ?
[...] (1,)
DELETE FROM users WHERE users.id = ?
[...] (5,)
SELECT count(*) AS count_1
FROM (SELECT users.id AS users_id,
                users.name AS users_name,
                users.fullname AS users_fullname,
                users.nickname AS users_nickname
FROM users
WHERE users.name = ?) AS anon_1
[...] ('jack',)
```

```python
>>> session.query(Address).filter(
...    Address.email_address.in_(['jack@google.com', 'j25@yahoo.com'])
... ).count()
0
```

{% hint style="info" %}
**Подробнее о каскадах**

Дополнительные сведения о конфигурации каскадов см. в разделе [Cascades](https://docs.sqlalchemy.org/en/14/orm/cascades.html#unitofwork-cascades). Каскадная функциональность также может легко интегрироваться с функциональностью **ON DELETE CASCADE** реляционной базы данных. Дополнительные сведения см. в разделе Использование каскада внешнего ключа ON DELETE с отношениями ORM ([Using foreign key ON DELETE cascade with ORM relationships](https://docs.sqlalchemy.org/en/14/orm/cascades.html#passive-deletes)).
{% endhint %}

## Построение отношения многие ко многим

Здесь мы переходим к бонусному раунду, но давайте продемонстрируем отношения «многие ко многим». Мы также добавим некоторые другие функции, просто для ознакомления. Мы сделаем наше приложение приложением для блога, где пользователи могут писать элементы **BlogPost**, с которыми связаны элементы ключевых слов **Keyword**.

Для простого «многие ко многим» нам нужно создать несопоставленную конструкцию таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), которая будет служить таблицей ассоциаций. Это выглядит следующим образом:

```python
>>> from sqlalchemy import Table, Text
>>> # ассоциативная таблица
>>> post_keywords = Table('post_keywords', Base.metadata,
...     Column('post_id', ForeignKey('posts.id'), primary_key=True),
...     Column('keyword_id', ForeignKey('keywords.id'), primary_key=True)
... )
```

Выше мы видим, что объявление таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) напрямую немного отличается от объявления сопоставленного класса. [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) — это функция-конструктор, поэтому каждый отдельный аргумент [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) отделяется запятой. Объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) также получает свое имя явно, а не берется из назначенного имени атрибута.

Затем мы определяем **BlogPost** и **Keyword**, используя взаимодополняющие конструкции [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), каждая из которых ссылается на таблицу _**post\_keywords**_ как на ассоциативную таблицу:

```python
>>> class BlogPost(Base):
...     __tablename__ = 'posts'
...
...     id = Column(Integer, primary_key=True)
...     user_id = Column(Integer, ForeignKey('users.id'))
...     headline = Column(String(255), nullable=False)
...     body = Column(Text)
...
...     # многие ко многим BlogPost<->Keyword
...     keywords = relationship('Keyword',
...                             secondary=post_keywords,
...                             back_populates='posts')
...
...     def __init__(self, headline, body, author):
...         self.author = author
...         self.headline = headline
...         self.body = body
...
...     def __repr__(self):
...         return "BlogPost(%r, %r, %r)" % (self.headline, self.body, self.author)


>>> class Keyword(Base):
...     __tablename__ = 'keywords'
...
...     id = Column(Integer, primary_key=True)
...     keyword = Column(String(50), nullable=False, unique=True)
...     posts = relationship('BlogPost',
...                          secondary=post_keywords,
...                          back_populates='keywords')
...
...     def __init__(self, keyword):
...         self.keyword = keyword
```

{% hint style="info" %}
Приведенные выше объявления классов иллюстрируют явные методы **\_\_init\_\_()**. Помните, что при использовании **Declarative** это необязательно!
{% endhint %}

Выше отношение «многие ко многим» — это `BlogPost.keywords`. Определяющей чертой отношения «многие ко многим» является аргумент ключевого слова **secondary**, который ссылается на объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), представляющий ассоциативную таблицу. Эта таблица содержит только столбцы, которые ссылаются на две стороны отношения; если у него есть какие-либо другие столбцы, такие как его собственный первичный ключ или внешние ключи к другим таблицам, SQLAlchemy требует другого шаблона использования, называемого «объектом ассоциации», описанного в разделе «Объект ассоциации» ([Association Object](https://docs.sqlalchemy.org/en/14/orm/basic\_relationships.html#association-pattern)).

Мы также хотели бы, чтобы наш класс **BlogPost** имел поле _**author**_. Мы добавим это как еще одно двунаправленное отношение, за исключением того, что у нас будет одна проблема: у одного пользователя может быть много сообщений в блоге. Когда мы обращаемся к `User.posts`, мы хотели бы иметь возможность дополнительно фильтровать результаты, чтобы не загружать всю коллекцию. Для этого мы используем параметр `lazy='dynamic'`, принятый функцией [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), который настраивает альтернативную **стратегию загрузки** для атрибута:

```python
>>> BlogPost.author = relationship(User, back_populates="posts")
>>> User.posts = relationship(BlogPost, back_populates="author", lazy="dynamic")
```

Создайте новые таблицы:

```python
>>> Base.metadata.create_all(engine)
```

```sql
BEGIN...
CREATE TABLE keywords (
    id INTEGER NOT NULL,
    keyword VARCHAR(50) NOT NULL,
    PRIMARY KEY (id),
    UNIQUE (keyword)
)
[...] ()
CREATE TABLE posts (
    id INTEGER NOT NULL,
    user_id INTEGER,
    headline VARCHAR(255) NOT NULL,
    body TEXT,
    PRIMARY KEY (id),
    FOREIGN KEY(user_id) REFERENCES users (id)
)
[...] ()
CREATE TABLE post_keywords (
    post_id INTEGER NOT NULL,
    keyword_id INTEGER NOT NULL,
    PRIMARY KEY (post_id, keyword_id),
    FOREIGN KEY(post_id) REFERENCES posts (id),
    FOREIGN KEY(keyword_id) REFERENCES keywords (id)
)
[...] ()
COMMIT
```

Использование не слишком отличается от того, что мы делали. Давайте дадим Венди несколько сообщений в блоге:

```python
>>> wendy = session.query(User).\
...                 filter_by(name='wendy').\
...                 one()
>>> post = BlogPost("Wendy's Blog Post", "This is a test", wendy)
>>> session.add(post)
```

```sql
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name = ?
[...] ('wendy',)
```

Мы сохраняем уникальные ключевые слова в базе данных, но мы знаем, что у нас их еще нет, поэтому мы можем просто создать их:

```python
>>> post.keywords.append(Keyword('wendy'))
>>> post.keywords.append(Keyword('firstpost'))
```

Теперь мы можем искать все сообщения в блогах по ключевому слову «firstpost». Мы будем использовать оператор **any** для поиска «сообщений в блоге, где любое из его ключевых слов имеет строку ключевого слова «firstpost»:

```python
>>> session.query(BlogPost).\
...             filter(BlogPost.keywords.any(keyword='firstpost')).\
...             all()
[BlogPost("Wendy's Blog Post", 'This is a test', <User(name='wendy', fullname='Wendy Williams', nickname='windy')>)]
```

```sql
INSERT INTO keywords (keyword) VALUES (?)
[...] ('wendy',)
INSERT INTO keywords (keyword) VALUES (?)
[...] ('firstpost',)
INSERT INTO posts (user_id, headline, body) VALUES (?, ?, ?)
[...] (2, "Wendy's Blog Post", 'This is a test')
INSERT INTO post_keywords (post_id, keyword_id) VALUES (?, ?)
[...] (...)
SELECT posts.id AS posts_id,
        posts.user_id AS posts_user_id,
        posts.headline AS posts_headline,
        posts.body AS posts_body
FROM posts
WHERE EXISTS (SELECT 1
    FROM post_keywords, keywords
    WHERE posts.id = post_keywords.post_id
        AND keywords.id = post_keywords.keyword_id
        AND keywords.keyword = ?)
[...] ('firstpost',)
```

Если мы хотим найти сообщения, принадлежащие пользователю _**wendy**_, мы можем указать запросу сузиться до этого объекта **User** в качестве родителя:

```python
>>> session.query(BlogPost).\
...             filter(BlogPost.author==wendy).\
...             filter(BlogPost.keywords.any(keyword='firstpost')).\
...             all()
[BlogPost("Wendy's Blog Post", 'This is a test', <User(name='wendy', fullname='Wendy Williams', nickname='windy')>)]
```

```sql
SELECT posts.id AS posts_id,
        posts.user_id AS posts_user_id,
        posts.headline AS posts_headline,
        posts.body AS posts_body
FROM posts
WHERE ? = posts.user_id AND (EXISTS (SELECT 1
    FROM post_keywords, keywords
    WHERE posts.id = post_keywords.post_id
        AND keywords.id = post_keywords.keyword_id
        AND keywords.keyword = ?))
[...] (2, 'firstpost')
```

Или мы можем использовать собственные отношения сообщений Венди, которые являются «динамическими» отношениями, чтобы запрашивать прямо оттуда:

```python
>>> wendy.posts.\
...         filter(BlogPost.keywords.any(keyword='firstpost')).\
...         all()
[BlogPost("Wendy's Blog Post", 'This is a test', <User(name='wendy', fullname='Wendy Williams', nickname='windy')>)]
```

```sql
SELECT posts.id AS posts_id,
        posts.user_id AS posts_user_id,
        posts.headline AS posts_headline,
        posts.body AS posts_body
FROM posts
WHERE ? = posts.user_id AND (EXISTS (SELECT 1
    FROM post_keywords, keywords
    WHERE posts.id = post_keywords.post_id
        AND keywords.id = post_keywords.keyword_id
        AND keywords.keyword = ?))
[...] (2, 'firstpost')
```

## Дополнительная ссылка

* ссылка на Query: [Query API](https://docs.sqlalchemy.org/en/14/orm/query.html)
* ссылка на Mapper: [Mapper Configuration](https://docs.sqlalchemy.org/en/14/orm/mapper\_config.html)
* ссылка на Relationship: [Relationship Configuration](https://docs.sqlalchemy.org/en/14/orm/relationships.html)
* ссылка на Session: [Using the Session](https://docs.sqlalchemy.org/en/14/orm/session.html)
