# Настройка соединения Join отношений

Отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) обычно создает соединение между двумя таблицами, исследуя отношение внешнего ключа между двумя таблицами, чтобы определить, какие столбцы следует сравнивать. Существует множество ситуаций, когда это поведение необходимо настроить.

## Обработка нескольких путей соединения

Одна из наиболее распространенных ситуаций, с которой приходится иметь дело, — это когда существует более одного пути внешнего ключа между двумя таблицами.

Рассмотрим класс **Customer**, который содержит два внешних ключа к классу **Address**:

```python
from sqlalchemy import Integer, ForeignKey, String, Column
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class Customer(Base):
    __tablename__ = 'customer'
    id = Column(Integer, primary_key=True)
    name = Column(String)

    billing_address_id = Column(Integer, ForeignKey("address.id"))
    shipping_address_id = Column(Integer, ForeignKey("address.id"))

    billing_address = relationship("Address")
    shipping_address = relationship("Address")

class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    street = Column(String)
    city = Column(String)
    state = Column(String)
    zip = Column(String)
```

Приведенное выше сопоставление, когда мы попытаемся его использовать, вызовет ошибку:

```bash
sqlalchemy.exc.AmbiguousForeignKeysError: Could not determine join
condition between parent/child tables on relationship
Customer.billing_address - there are multiple foreign key
paths linking the tables.  Specify the 'foreign_keys' argument,
providing a list of those columns which should be
counted as containing a foreign key reference to the parent table.
```

Сообщение выше довольно длинное. Есть много потенциальных сообщений, которые может возвращать функция [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), которые были тщательно адаптированы для обнаружения различных распространенных проблем с конфигурацией; большинство из них предложит дополнительную конфигурацию, необходимую для устранения неоднозначности или другой отсутствующей информации.

В этом случае сообщение хочет, чтобы мы уточнили **каждое** отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), указав для **каждого из них**, какой столбец внешнего ключа следует учитывать, и соответствующая форма выглядит следующим образом:

```python
class Customer(Base):
    __tablename__ = 'customer'
    id = Column(Integer, primary_key=True)
    name = Column(String)

    billing_address_id = Column(Integer, ForeignKey("address.id"))
    shipping_address_id = Column(Integer, ForeignKey("address.id"))

    billing_address = relationship("Address", foreign_keys=[billing_address_id])
    shipping_address = relationship("Address", foreign_keys=[shipping_address_id])
```

Выше мы указываем аргумент **foreign\_keys**, который представляет собой столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) или список объектов столбцов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), указывающих, что эти столбцы следует считать «чужими», или, другими словами, столбцы, содержащие значение, относящееся к родительской таблице. При загрузке отношения **Customer.billing\_address** из объекта **Customer** будет использоваться значение, присутствующее в **billing\_address\_id**, для идентификации загружаемой строки **Address**; аналогично, **shipping\_address\_id** используется для отношения **shipping\_address**. Связь двух столбцов также играет роль во время персистентности; вновь сгенерированный первичный ключ только что вставленного объекта **Address** будет скопирован в соответствующий столбец внешнего ключа связанного объекта **Customer** во время сброса.

При указании **foreign\_keys** с помощью **Declarative** мы также можем использовать имена строк для указания, однако важно, чтобы при использовании списка **список был частью строки**:

```python
billing_address = relationship(
   "Address",
   foreign_keys="[Customer.billing_address_id]"
)
```

В этом конкретном примере список не нужен в любом случае, так как нам нужен только один столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column):

```python
billing_address = relationship(
    "Address",
    foreign_keys="Customer.billing_address_id"
)
```

{% hint style="danger" %}
При передаче в виде строки, которую может вычислить Python, аргумент [relationship.foreign\_keys](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.foreign\_keys) интерпретируется с помощью функции Python **eval()**. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ**. Подробную информацию о декларативной оценке аргументов отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

## Указание альтернативных условий объединения Join

Поведение отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) по умолчанию при построении соединения заключается в том, что оно приравнивает значение столбцов первичного ключа с одной стороны к значению столбцов, ссылающихся на внешний ключ, с другой. Мы можем изменить этот критерий на любой, какой захотим, с помощью аргумента [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin), а также аргумента [relationship.secondaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondaryjoin) в случае, когда используется «вторичная» таблица.

В приведенном ниже примере, используя класс **User**, а также класс **Address**, в котором хранится почтовый адрес, мы создаем отношение **boston\_addresses**, которое будет загружать только те объекты **Address**, которые указывают город «Бостон»:

```python
from sqlalchemy import Integer, ForeignKey, String, Column
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    name = Column(String)
    boston_addresses = relationship("Address",
                    primaryjoin="and_(User.id==Address.user_id, "
                        "Address.city=='Boston')")

class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('user.id'))

    street = Column(String)
    city = Column(String)
    state = Column(String)
    zip = Column(String)
```

В этом строковом SQL-выражении мы использовали конструкцию соединения [and\_()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.and\_), чтобы установить два различных предиката для условия соединения — соединение столбцов **User.id** и **Address.user\_id** друг с другом, а также ограничение строк в **Address** только `city='Boston'`. При использовании декларативного типа элементарные функции SQL, такие как [and\_()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.and\_), автоматически доступны в оцениваемом пространстве имен аргумента строкового отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship).

{% hint style="danger" %}
При передаче в виде строки, которую может вычислить Python, аргумент [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) интерпретируется с помощью функции Python **eval()**. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ**. Подробную информацию о декларативной оценке аргументов отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

Пользовательские критерии, которые мы используем в [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) обычно имеют значение только тогда, когда SQLAlchemy отображает SQL для загрузки или представления этой связи. То есть он используется в операторе SQL, который создается для выполнения ленивой загрузки для каждого атрибута, или когда соединение создается во время запроса, например, с помощью [Query.join()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.join), или с помощью нетерпеливых «joined» или «subquery» стилей загрузки. При манипулировании объектами в памяти мы можем поместить любой объект **Address** в коллекцию **boston\_addresses**, независимо от значения атрибута `.city`. Объекты останутся в коллекции до тех пор, пока срок действия атрибута не истечет и он не будет повторно загружен из базы данных, к которой применяется критерий. Когда происходит сброс, объекты внутри **boston\_addresses** будут сброшены безоговорочно, присваивая значение столбца **user.id** первичного ключа столбцу **address.user\_id**, содержащему внешний ключ, для каждой строки. Критерии города _**city**_ здесь не действуют, так как процесс сброса заботится только о синхронизации значений первичного ключа со ссылками на значения внешнего ключа.

## Создание пользовательских внешних условий

Еще одним элементом условия первичного соединения является то, как определяются столбцы, которые считаются «чужими». Обычно некоторое подмножество объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) задает [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey) или иным образом является частью [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint), относящейся к условию соединения. Отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) смотрит на этот статус внешнего ключа, поскольку он решает, как он должен загружать и сохранять данные для этого отношения. Однако аргумент [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) можно использовать для создания условия соединения, которое не включает внешние ключи уровня «схемы». Мы можем комбинировать [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) с [relationship.foreign\_keys](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.foreign\_keys) и [relationship.remote\_side](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.remote\_side) явно, чтобы установить такое соединение.

Ниже класс **HostEntry** присоединяется к самому себе, приравнивая столбец строкового содержимого **content** к столбцу **ip\_address**, который является типом PostgreSQL, называемым **INET**. Нам нужно использовать [cast()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.cast), чтобы привести одну сторону соединения к типу другой:

```python
from sqlalchemy import cast, String, Column, Integer
from sqlalchemy.orm import relationship
from sqlalchemy.dialects.postgresql import INET

from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class HostEntry(Base):
    __tablename__ = 'host_entry'

    id = Column(Integer, primary_key=True)
    ip_address = Column(INET)
    content = Column(String(50))

    # relationship() используя явное foreign_keys, remote_side
    parent_host = relationship("HostEntry",
                        primaryjoin=ip_address == cast(content, INET),
                        foreign_keys=content,
                        remote_side=ip_address
                    )
```

Вышеупомянутое отношение создаст соединение, подобное:

```sql
SELECT host_entry.id, host_entry.ip_address, host_entry.content
FROM host_entry JOIN host_entry AS host_entry_1
ON host_entry_1.ip_address = CAST(host_entry.content AS INET)
```

Синтаксис, альтернативный приведенному выше, заключается в использовании [аннотаций](https://docs.sqlalchemy.org/en/14/glossary.html#term-annotations) [foreign()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.foreign) и [remote()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.remote), встроенных в выражение [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin). Этот синтаксис представляет собой аннотации, которые [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) обычно применяет сама по себе к условию соединения с учетом аргументов [relationship.foreign\_keys](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.foreign\_keys) и [relationship.remote\_side](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.remote\_side). Эти функции могут быть более краткими, когда присутствует явное условие соединения, и дополнительно служат для пометки именно того столбца, который является «чужим» или «удаленным», независимо от того, указан ли этот столбец несколько раз или в сложных выражениях SQL:

```python
from sqlalchemy.orm import foreign, remote

class HostEntry(Base):
    __tablename__ = 'host_entry'

    id = Column(Integer, primary_key=True)
    ip_address = Column(INET)
    content = Column(String(50))

    # relationship() использует явные аннотации foreign() и remote()
    # вместо отдельных аргументов
    parent_host = relationship("HostEntry",
                        primaryjoin=remote(ip_address) == \
                                cast(foreign(content), INET),
                    )
```

## Использование пользовательских операторов в условиях соединения

Другим вариантом использования отношений является использование пользовательских операторов, таких как оператор PostgreSQL «содержится внутри» **`<<`** при объединении с такими типами, как [**INET**](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html#sqlalchemy.dialects.postgresql.INET) и [**CIDR**](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html#sqlalchemy.dialects.postgresql.CIDR). Для пользовательских операторов мы используем функцию [Operators.op()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.Operators.op):

```python
inet_column.op("<<")(cidr_column)
```

Однако, если мы создадим [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) с помощью этого оператора, для отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) все равно потребуется дополнительная информация. Это связано с тем, что когда он проверяет наше условие первичного соединения, он специально ищет операторы, **используемые для сравнения**, и обычно это фиксированный список, содержащий известные операторы сравнения, такие как `==`, `<` и т. д. Таким образом, для участия нашего пользовательского оператора в этой системе нам нужно, чтобы он зарегистрировался как оператор сравнения с помощью параметра [Operators.op.is\_comparison](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.Operators.op.params.is\_comparison):

```python
inet_column.op("<<", is_comparison=True)(cidr_column)
```

Полный пример:

```python
class IPA(Base):
    __tablename__ = 'ip_address'

    id = Column(Integer, primary_key=True)
    v4address = Column(INET)

    network = relationship("Network",
                        primaryjoin="IPA.v4address.op('<<', is_comparison=True)"
                            "(foreign(Network.v4representation))",
                        viewonly=True
                    )
class Network(Base):
    __tablename__ = 'network'

    id = Column(Integer, primary_key=True)
    v4representation = Column(CIDR)
```

Выше запрос типа:

```python
session.query(IPA).join(IPA.network)
```

Будет отображаться как:

```sql
SELECT ip_address.id AS ip_address_id, ip_address.v4address AS ip_address_v4address
FROM ip_address JOIN network ON ip_address.v4address << network.v4representation
```

{% hint style="warning" %}
**Новое в версии 0.9.2**: — Добавлен флаг [Operators.op.is\_comparison](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.Operators.op.params.is\_comparison) для облегчения создания конструкций [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) с использованием пользовательских операторов.
{% endhint %}

## Пользовательские операторы на основе функций SQL

Вариант использования для [Operators.op.is\_comparison](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.Operators.op.params.is\_comparison) — это когда мы используем не оператор, а функцию SQL. Типичным примером этого варианта использования являются функции PostgreSQL **PostGIS**, однако может применяться любая функция SQL в любой базе данных, которая преобразуется в двоичное условие. Чтобы соответствовать этому варианту использования, метод [FunctionElement.as\_comparison()](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.as\_comparison) может изменить любую функцию SQL, например те, которые вызываются из пространства имен [func](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.func), чтобы указать ORM, что функция производит сравнение двух выражений. Пример ниже иллюстрирует это с библиотекой [Geoalchemy2](https://geoalchemy-2.readthedocs.io):

```python
from geoalchemy2 import Geometry
from sqlalchemy import Column, Integer, func
from sqlalchemy.orm import relationship, foreign

class Polygon(Base):
    __tablename__ = "polygon"
    id = Column(Integer, primary_key=True)
    geom = Column(Geometry("POLYGON", srid=4326))
    points = relationship(
        "Point",
        primaryjoin="func.ST_Contains(foreign(Polygon.geom), Point.geom).as_comparison(1, 2)",
        viewonly=True,
    )

class Point(Base):
    __tablename__ = "point"
    id = Column(Integer, primary_key=True)
    geom = Column(Geometry("POINT", srid=4326))
```

Выше [FunctionElement.as\_comparison()](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.as\_comparison) указывает, что SQL-функция **func.ST\_Contains()** сравнивает выражения **Polygon.geom** и **Point.geom**. Аннотация [foreign()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.foreign) дополнительно отмечает, какой столбец берет на себя роль «внешнего ключа» в данном конкретном отношении.

{% hint style="warning" %}
**Новое в версии 1.3**: Добавлен [FunctionElement.as\_comparison()](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.as\_comparison).
{% endhint %}

## Перекрывающиеся внешние ключи

Редкий сценарий может возникнуть, когда используются составные внешние ключи, например, один столбец может быть предметом более чем одного столбца, на который ссылается ограничение внешнего ключа.

Рассмотрим (по общему признанию, сложное) отображение, такое как объект **Magazine**, на который ссылается как объект **Writer**, так и объект **Article**, использующий составную схему первичного ключа, которая включает **magazine\_id** для обоих; затем, чтобы статья **Article** также ссылалась на **Writer**, **Article.magazine\_id** участвует в двух отдельных отношениях; **Article.magazine** и **Article.writer**:

```python
class Magazine(Base):
    __tablename__ = 'magazine'

    id = Column(Integer, primary_key=True)


class Article(Base):
    __tablename__ = 'article'

    article_id = Column(Integer)
    magazine_id = Column(ForeignKey('magazine.id'))
    writer_id = Column()

    magazine = relationship("Magazine")
    writer = relationship("Writer")

    __table_args__ = (
        PrimaryKeyConstraint('article_id', 'magazine_id'),
        ForeignKeyConstraint(
            ['writer_id', 'magazine_id'],
            ['writer.id', 'writer.magazine_id']
        ),
    )


class Writer(Base):
    __tablename__ = 'writer'

    id = Column(Integer, primary_key=True)
    magazine_id = Column(ForeignKey('magazine.id'), primary_key=True)
    magazine = relationship("Magazine")
```

Когда приведенное выше сопоставление настроено, мы увидим это предупреждение:

```bash
SAWarning: relationship 'Article.writer' will copy column
writer.magazine_id to column article.magazine_id,
which conflicts with relationship(s): 'Article.magazine'
(copies magazine.id to article.magazine_id). Consider applying
viewonly=True to read-only relationships, or provide a primaryjoin
condition marking writable columns with the foreign() annotation.
```

То, к чему это относится, происходит из-за того, что **Article.magazine\_id** является предметом двух разных ограничений внешнего ключа; он ссылается непосредственно на **Magazine.id** как на исходный столбец, но также ссылается на **Writer.magazine\_id** как на исходный столбец в контексте составного ключа для **Writer**. Если мы свяжем статью **Article** с определенным журналом **Magazine**, но затем свяжем статью **Article** с писателем **Writer**, который связан с другим журналом **Writer**, ORM недетерминистически перезапишет **Article.magazine\_id**, незаметно изменив журнал **Magazine**, на который мы ссылаемся; он также может попытаться поместить **NULL** в этот столбец, если мы отключим **Writer** от статьи **Article**. Предупреждение сообщает нам, что это так.

Чтобы решить эту проблему, нам нужно изменить поведение статьи **Article**, чтобы включить все три следующие функции:

1. Статья **Article** в первую очередь пишет в **Article.magazine\_id** на основе данных, сохраняемых только в отношении **Article.magazine**, т. е. значения, скопированного из **Magazine.id**.
2. Статья **Article** может писать в **Article.writer\_id** от имени данных, сохраняемых в отношении **Article.writer**, но только в столбце **Writer.id**; столбец **Writer.magazine\_id** не следует записывать в **Article.magazine\_id**, поскольку в конечном итоге он берется из **Magazine.id**.
3. Статья **Article** принимает во внимание **Article.magazine\_id** при загрузке **Article.writer**, даже если он _**не пишет**_ в нее от имени этого отношения.

Чтобы получить только **#1** и **#2**, мы могли бы указать только **Article.writer\_id** в качестве «внешних ключей» для **Article.writer**:

```python
class Article(Base):
    # ...

    writer = relationship("Writer", foreign_keys='Article.writer_id')
```

Однако это приводит к тому, что **Article.writer** не принимает во внимание **Article.magazine\_id** при запросе **Writer**:

```sql
SELECT article.article_id AS article_article_id,
    article.magazine_id AS article_magazine_id,
    article.writer_id AS article_writer_id
FROM article
JOIN writer ON writer.id = article.writer_id
```

Таким образом, чтобы получить все **#1**, **#2** и **#3**, мы выражаем условие объединения, а также какие столбцы должны быть записаны, комбинацией [relationship.primatyjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) полностью,  вместе с аргументом [relationship.foreign\_keys](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.foreign\_keys) или более кратко, аннотируя с помощью [foreign()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.foreign):

```python
class Article(Base):
    # ...

    writer = relationship(
        "Writer",
        primaryjoin="and_(Writer.id == foreign(Article.writer_id), "
                    "Writer.magazine_id == Article.magazine_id)")
```

{% hint style="warning" %}
**Изменено в версии 1.0.0**: ORM попытается предупредить, когда столбец используется в качестве цели синхронизации из более чем одного отношения одновременно.
{% endhint %}

## Нереляционные сравнения/материализованный путь

{% hint style="danger" %}
в этом разделе подробно описана **экспериментальная** функция.
{% endhint %}

Использование пользовательских выражений означает, что мы можем создавать неортодоксальные условия соединения, которые не подчиняются обычной модели первичного/внешнего ключа. Одним из таких примеров является шаблон материализованного пути, в котором мы сравниваем строки на наличие перекрывающихся токенов пути, чтобы создать древовидную структуру.

Осторожно используя функции [foreign()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.foreign) и [remote()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.remote), мы можем построить отношение, которое эффективно создаст рудиментарную материализованную систему путей. По существу, когда [foreign()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.foreign) и [remote()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.remote) находятся на одной стороне выражения сравнения, связь рассматривается как «один ко многим»; когда они находятся _**по разные стороны**_, отношение считается «многие к одному». Для сравнения, которое мы будем использовать здесь, мы будем иметь дело с коллекциями, поэтому мы сохраняем настройки как «один ко многим»:

```python
class Element(Base):
    __tablename__ = 'element'

    path = Column(String, primary_key=True)

    descendants = relationship('Element',
                           primaryjoin=
                                remote(foreign(path)).like(
                                        path.concat('/%')),
                           viewonly=True,
                           order_by=path)
```

Выше, если задан объект **Element** с атрибутом пути `«/foo/bar2»`, мы ищем, чтобы загрузка **Element.descendants** выглядела так:

```sql
SELECT element.path AS element_path
FROM element
WHERE element.path LIKE ('/foo/bar2' || '/%') ORDER BY element.path
```

{% hint style="warning" %}
**Новое в версии 0.9.5**: добавлена поддержка, позволяющая проводить сравнение одного столбца с самим собой в условиях первичного соединения, а также для условий первичного соединения, использующих [ColumnOperators.like()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.ColumnOperators.like) в качестве оператора сравнения.
{% endhint %}

## Самореферентная связь «многие ко многим»

{% hint style="info" %}
Смотри также:

В этом разделе описан двухтабличный вариант шаблона «список смежности», описанный в разделе [Отношения списка смежности](otnosheniya-spiska-smezhnosti.md). Обязательно ознакомьтесь с шаблонами самореферентных запросов в подразделах «[Стратегии самореферентных запросов](otnosheniya-spiska-smezhnosti.md#strategii-samoreferentnykh-zaprosov)» и «[Настройка самореферентной нетерпеливой загрузки](otnosheniya-spiska-smezhnosti.md#nastroika-samoreferentnoi-zhadnoi-zagruzki)», которые в равной степени применимы к обсуждаемому здесь шаблону сопоставления.
{% endhint %}

Отношения «многие ко многим» могут быть настроены с помощью одного или обоих из отношения «[relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin)» и «[relationship.secondaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondaryjoin)» — последнее важно для отношения, которое указывает ссылку «многие ко многим» с использованием аргумента «[relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary)». Обычная ситуация, связанная с использованием [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) и [relationship.secondaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondaryjoin), возникает при установлении отношения «многие ко многим» между классом и самим собой, как показано ниже:

```python
from sqlalchemy import Integer, ForeignKey, String, Column, Table
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

node_to_node = Table("node_to_node", Base.metadata,
    Column("left_node_id", Integer, ForeignKey("node.id"), primary_key=True),
    Column("right_node_id", Integer, ForeignKey("node.id"), primary_key=True)
)

class Node(Base):
    __tablename__ = 'node'
    id = Column(Integer, primary_key=True)
    label = Column(String)
    right_nodes = relationship("Node",
                        secondary=node_to_node,
                        primaryjoin=id==node_to_node.c.left_node_id,
                        secondaryjoin=id==node_to_node.c.right_node_id,
                        backref="left_nodes"
    )
```

Где выше, SQLAlchemy не может автоматически знать, какие столбцы должны соединяться с какими для отношений **right\_nodes** и **left\_nodes**. Аргументы [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) и [relationship.secondaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondaryjoin) определяют, как мы хотели бы присоединиться к таблице ассоциаций. В декларативной форме выше, когда мы объявляем эти условия в блоке Python, который соответствует классу **Node**, переменная **id** доступна непосредственно как объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), к которому мы хотим присоединиться.

В качестве альтернативы мы можем определить аргументы [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) и [relationship.secondaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondaryjoin), используя строки, что подходит в случае, если в нашей конфигурации еще нет доступного объекта столбца **Node.id** или таблицы **node\_to\_node**, возможно, еще недоступна. При обращении к простому объекту [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) в декларативной строке мы используем строковое имя таблицы, поскольку оно присутствует в метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData):

```python
class Node(Base):
    __tablename__ = 'node'
    id = Column(Integer, primary_key=True)
    label = Column(String)
    right_nodes = relationship("Node",
                        secondary="node_to_node",
                        primaryjoin="Node.id==node_to_node.c.left_node_id",
                        secondaryjoin="Node.id==node_to_node.c.right_node_id",
                        backref="left_nodes"
    )
```

{% hint style="danger" %}
При передаче в виде строки, которую может вычислить Python, аргументы [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) и [relationship.secondaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondaryjoin) интерпретируются с использованием функции Python **`eval()`**. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТИ СТРОКИ**. Подробную информацию о декларативной оценке аргументов отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

Здесь аналогична классическая ситуация сопоставления, когда **node\_to\_node** может быть присоединен к **node.c.id**:

```python
from sqlalchemy import Integer, ForeignKey, String, Column, Table, MetaData
from sqlalchemy.orm import relationship, registry

metadata_obj = MetaData()
mapper_registry = registry()

node_to_node = Table("node_to_node", metadata_obj,
    Column("left_node_id", Integer, ForeignKey("node.id"), primary_key=True),
    Column("right_node_id", Integer, ForeignKey("node.id"), primary_key=True)
)

node = Table("node", metadata_obj,
    Column('id', Integer, primary_key=True),
    Column('label', String)
)
class Node(object):
    pass

mapper_registry.map_imperatively(Node, node, properties={
    'right_nodes':relationship(Node,
                        secondary=node_to_node,
                        primaryjoin=node.c.id==node_to_node.c.left_node_id,
                        secondaryjoin=node.c.id==node_to_node.c.right_node_id,
                        backref="left_nodes"
                    )})
```

Обратите внимание, что в обоих примерах ключевое слово [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) указывает обратную ссылку **left\_nodes** — когда отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) создает второе отношение в обратном направлении, оно достаточно умно, чтобы поменять местами аргументы [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) и [relationship.secondaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondaryjoin).

{% hint style="info" %}
Смотри также:

* [Отношения списка смежности](otnosheniya-spiska-smezhnosti.md#otnosheniya-spiska-smezhnosti) — версия с одной таблицей
* [Стратегии самореферентных запросов](otnosheniya-spiska-smezhnosti.md#strategii-samoreferentnykh-zaprosov) — советы по запросам с самореферентными сопоставлениями
* [Настройка самореферентной жадной загрузки](otnosheniya-spiska-smezhnosti.md#nastroika-samoreferentnoi-zhadnoi-zagruzki) — советы по жадной загрузке с самореферентным сопоставлением
{% endhint %}

## Композитные «вторичные» соединения

{% hint style="info" %}
В этом разделе представлены крайние случаи, которые частично поддерживаются SQLAlchemy, однако рекомендуется по возможности решать подобные проблемы более простыми способами, используя разумные реляционные макеты и/или [атрибуты Python](../konfiguraciya-mapper/otobrazhenie-kolonok-i-vyrazhenii/izmenenie-povedeniya-atributov.md#ispolzovanie-deskriptorov-i-gibridov).
{% endhint %}

Иногда, когда кто-то пытается построить отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) между двумя таблицами, необходимо, чтобы для их объединения было задействовано более двух или трех таблиц. Это область отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), где каждый стремится раздвинуть границы возможного, и часто окончательное решение для многих из этих экзотических вариантов использования должно быть выработано в списке рассылки SQLAlchemy.

В более поздних версиях SQLAlchemy в некоторых из этих случаев можно использовать параметр [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary), чтобы предоставить составную цель, состоящую из нескольких таблиц. Ниже приведен пример такого условия соединения (требуется версия не ниже 0.9.2, чтобы функционировать как есть):

```python
class A(Base):
    __tablename__ = 'a'

    id = Column(Integer, primary_key=True)
    b_id = Column(ForeignKey('b.id'))

    d = relationship("D",
                secondary="join(B, D, B.d_id == D.id)."
                            "join(C, C.d_id == D.id)",
                primaryjoin="and_(A.b_id == B.id, A.id == C.a_id)",
                secondaryjoin="D.id == B.d_id",
                uselist=False,
                viewonly=True
                )

class B(Base):
    __tablename__ = 'b'

    id = Column(Integer, primary_key=True)
    d_id = Column(ForeignKey('d.id'))

class C(Base):
    __tablename__ = 'c'

    id = Column(Integer, primary_key=True)
    a_id = Column(ForeignKey('a.id'))
    d_id = Column(ForeignKey('d.id'))

class D(Base):
    __tablename__ = 'd'

    id = Column(Integer, primary_key=True)
```

В приведенном выше примере мы предоставляем все три из [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary), [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) и [relationship.secondaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondaryjoin) в декларативном стиле, относящемся непосредственно к именованным таблицам **a**, **b**, **c**, **d**. Запрос от **A** к **D** выглядит так:

```python
sess.query(A).join(A.d).all()
```

```sql
SELECT a.id AS a_id, a.b_id AS a_b_id
FROM a JOIN (
    b AS b_1 JOIN d AS d_1 ON b_1.d_id = d_1.id
        JOIN c AS c_1 ON c_1.d_id = d_1.id)
    ON a.b_id = b_1.id AND a.id = c_1.a_id JOIN d ON d.id = b_1.d_id
```

В приведенном выше примере мы воспользовались возможностью поместить несколько таблиц во «вторичный» контейнер, чтобы мы могли объединяться между многими таблицами, сохраняя при этом «простоту» для отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), поскольку есть только «одна» таблица. как с «левой», так и с «правой» стороны; сложность держится в середине.

{% hint style="danger" %}
Связь, подобная приведенной выше, обычно помечается как **`viewonly=True`** и должна рассматриваться как доступная _**только для чтения**_. Хотя иногда существуют способы сделать отношения, подобные приведенным выше, доступными для записи, это, как правило, сложно и подвержено ошибкам.
{% endhint %}

## Отношение к псевдониму класса

{% hint style="warning" %}
**Новое в версии 1.3**: конструкция [AliasedClass](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.util.AliasedClass) теперь может быть указана в качестве цели отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), заменяя предыдущий подход с использованием неосновных преобразователей, которые имели ограничения, такие как то, что они также не наследуют суботношения отображаемого объекта. поскольку они требовали сложной конфигурации по сравнению с альтернативным выбором. Рецепты в этом разделе теперь обновлены для использования [AliasedClass](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.util.AliasedClass).
{% endhint %}

В предыдущем разделе мы проиллюстрировали метод, в котором мы использовали [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary) для размещения дополнительных таблиц в условиях соединения. Есть один сложный случай соединения, когда даже этого метода недостаточно; когда мы пытаемся соединиться от **A** к **B**, используя любое количество **C**, **D** и т. д. между ними, однако есть также условия соединения _**непосредственно**_ между **A** и **B**. В этом случае соединение от **A** к **B** может быть трудно выразить только с помощью сложного условия [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin), поскольку промежуточные таблицы могут нуждаться в специальной обработке, и это также не может быть выражено с помощью объекта [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary), поскольку шаблон **A->secondary->B** не поддерживает никаких прямых ссылок между **A** и **B**. Когда возникает этот _**чрезвычайно сложный случай**_, мы можем прибегнуть к созданию второго сопоставления в качестве цели для отношения. Именно здесь мы используем [AliasedClass](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.util.AliasedClass), чтобы сделать сопоставление с классом, который включает все дополнительные таблицы, необходимые для этого объединения. Чтобы создать этот преобразователь в качестве «альтернативного» отображения для нашего класса, мы используем функцию [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) для создания новой конструкции, а затем применяем отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) к объекту, как если бы это был простой отображаемый класс.

Ниже показано отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) с простым соединением от **A** к **B**, однако условие первичного соединения дополнено двумя дополнительными объектами **C** и **D**, которые также должны иметь строки, которые одновременно совпадают со строками в **A** и **B**:

```python
class A(Base):
    __tablename__ = 'a'

    id = Column(Integer, primary_key=True)
    b_id = Column(ForeignKey('b.id'))

class B(Base):
    __tablename__ = 'b'

    id = Column(Integer, primary_key=True)

class C(Base):
    __tablename__ = 'c'

    id = Column(Integer, primary_key=True)
    a_id = Column(ForeignKey('a.id'))

    some_c_value = Column(String)

class D(Base):
    __tablename__ = 'd'

    id = Column(Integer, primary_key=True)
    c_id = Column(ForeignKey('c.id'))
    b_id = Column(ForeignKey('b.id'))

    some_d_value = Column(String)

# 1. установить join() как переменную, поэтому
# мы можем ссылаться на него в отображении несколько раз.
j = join(B, D, D.b_id == B.id).join(C, C.id == D.c_id)

# 2. создать AliasedClass к B
B_viacd = aliased(B, j, flat=True)

A.b = relationship(B_viacd, primaryjoin=A.b_id == j.c.b_id)
```

С приведенным выше сопоставлением простое соединение выглядит так:

```python
sess.query(A).join(A.b).all()
```

```sql
SELECT a.id AS a_id, a.b_id AS a_b_id
FROM a JOIN (b JOIN d ON d.b_id = b.id JOIN c ON c.id = d.c_id) ON a.b_id = b.id
```

### Использование цели AliasedClass в запросах

В предыдущем примере отношение **A.b** относится к объекту **B\_viacd** как к цели, а **не** непосредственно к классу **B**. Чтобы добавить дополнительные критерии, связанные с отношением **A.b**, обычно необходимо напрямую ссылаться на **B\_viacd**, а не использовать **B**, особенно в случае, когда целевой объект **A.b** должен быть преобразован в псевдоним или подзапрос. Ниже показано то же отношение с использованием подзапроса, а не соединения:

```python
subq = select(B).join(D, D.b_id == B.id).join(C, C.id == D.c_id).subquery()

B_viacd_subquery = aliased(B, subq)

A.b = relationship(B_viacd_subquery, primaryjoin=A.b_id == subq.c.id)
```

Запрос, использующий указанное выше отношение **A.b**, отобразит подзапрос:

```python
sess.query(A).join(A.b).all()
```

```sql
SELECT a.id AS a_id, a.b_id AS a_b_id
FROM a JOIN (SELECT b.id AS id, b.some_b_column AS some_b_column
FROM b JOIN d ON d.b_id = b.id JOIN c ON c.id = d.c_id) AS anon_1 ON a.b_id = anon_1.id
```

Если мы хотим добавить дополнительные критерии на основе соединения **A.b**, мы должны сделать это с точки зрения **B\_viacd\_subquery**, а не напрямую **B**:

```python
(
  sess.query(A).join(A.b).
  filter(B_viacd_subquery.some_b_column == "some b").
  order_by(B_viacd_subquery.id)
).all()
```

```sql
SELECT a.id AS a_id, a.b_id AS a_b_id
FROM a JOIN (SELECT b.id AS id, b.some_b_column AS some_b_column
FROM b JOIN d ON d.b_id = b.id JOIN c ON c.id = d.c_id) AS anon_1 ON a.b_id = anon_1.id
WHERE anon_1.some_b_column = ? ORDER BY anon_1.id
```

## Отношения, ограниченные строками, с оконными функциями

Еще одним интересным вариантом использования отношений с объектами [AliasedClass](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.util.AliasedClass) являются ситуации, когда отношение должно быть присоединено к специализированному **SELECT** любой формы. Один сценарий — это когда желательно использовать оконную функцию, например, чтобы ограничить количество строк, которые должны быть возвращены для отношения. Пример ниже иллюстрирует отношение неосновного преобразователя, которое будет загружать первые десять элементов для каждой коллекции:

```python
class A(Base):
    __tablename__ = 'a'

    id = Column(Integer, primary_key=True)


class B(Base):
    __tablename__ = 'b'
    id = Column(Integer, primary_key=True)
    a_id = Column(ForeignKey("a.id"))

partition = select(
    B,
    func.row_number().over(
        order_by=B.id, partition_by=B.a_id
    ).label('index')
).alias()

partitioned_b = aliased(B, partition)

A.partitioned_bs = relationship(
    partitioned_b,
    primaryjoin=and_(partitioned_b.a_id == A.id, partition.c.index < 10)
)
```

Мы можем использовать приведенную выше связь **partitioned\_bs** с большинством стратегий загрузчика, например, с [selectinload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.selectinload):

```
for a1 in s.query(A).options(selectinload(A.partitioned_bs)):
    print(a1.partitioned_bs)   # <-- будет не более десяти объектов
```

Где выше запрос **«selectinload»** выглядит так:

```sql
SELECT
    a_1.id AS a_1_id, anon_1.id AS anon_1_id, anon_1.a_id AS anon_1_a_id,
    anon_1.data AS anon_1_data, anon_1.index AS anon_1_index
FROM a AS a_1
JOIN (
    SELECT b.id AS id, b.a_id AS a_id, b.data AS data,
    row_number() OVER (PARTITION BY b.a_id ORDER BY b.id) AS index
    FROM b) AS anon_1
ON anon_1.a_id = a_1.id AND anon_1.index < %(index_1)s
WHERE a_1.id IN ( ... primary key collection ...)
ORDER BY a_1.id
```

Выше для каждого совпадающего первичного ключа в **«a»** мы получим первые десять **«bs»** в порядке, указанном **«b.id»**. Разбивая по **«a\_id»**, мы гарантируем, что каждый «номер строки» является локальным по отношению к родительскому **«a\_id»**.

Такое сопоставление обычно также включает «простую» связь от **«А»** к **«В»** для операций постоянства, а также когда требуется полный набор объектов **«В»** для **«А»**.

## Создание свойств с поддержкой запросов

Очень амбициозные настраиваемые условия соединения могут не сохраняться напрямую, а в некоторых случаях могут даже загружаться неправильно. Чтобы удалить часть уравнения, связанную с постоянством, используйте флаг [relationship.viewonly](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.viewonly) для отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), который устанавливает его как атрибут только для чтения (данные, записываемые в коллекцию, будут игнорироваться при **flush()**). Однако в крайних случаях рассмотрите возможность использования обычного свойства Python в сочетании с [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) следующим образом:

```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)

    @property
    def addresses(self):
        return object_session(self).query(Address).with_parent(self).filter(...).all()
```

В других случаях дескриптор может быть создан для использования существующих данных в Python. См. раздел «[Использование дескрипторов и гибридов](../konfiguraciya-mapper/otobrazhenie-kolonok-i-vyrazhenii/izmenenie-povedeniya-atributov.md#ispolzovanie-deskriptorov-i-gibridov)» для более общего обсуждения специальных атрибутов Python.

{% hint style="info" %}
Смотри также:

[Using Descriptors and Hybrids](../konfiguraciya-mapper/otobrazhenie-kolonok-i-vyrazhenii/izmenenie-povedeniya-atributov.md#ispolzovanie-deskriptorov-i-gibridov)
{% endhint %}
