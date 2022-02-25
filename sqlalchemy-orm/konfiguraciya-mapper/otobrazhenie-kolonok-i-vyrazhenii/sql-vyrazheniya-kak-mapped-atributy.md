# SQL выражения как mapped атрибуты

Атрибуты сопоставленного класса можно связать с выражениями SQL, которые можно использовать в запросах.

## Использование гибрида

Самый простой и гибкий способ связать относительно простые SQL-выражения с классом — использовать так называемый «гибридный атрибут», описанный в разделе «[Гибридные атрибуты](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html)». Гибрид обеспечивает выражение, которое работает как на уровне Python, так и на уровне выражения SQL. Например, ниже мы сопоставляем класс **User**, содержащий атрибуты **firstname** и **lastname**, и включаем гибрид, который предоставит нам полное имя, которое представляет собой конкатенацию двух строк:

```python
from sqlalchemy.ext.hybrid import hybrid_property

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    firstname = Column(String(50))
    lastname = Column(String(50))

    @hybrid_property
    def fullname(self):
        return self.firstname + " " + self.lastname
```

Выше атрибут **fullname** интерпретируется как на уровне экземпляра, так и на уровне класса, поэтому он доступен из экземпляра:

```python
some_user = session.query(User).first()
print(some_user.fullname)
```

а также может использоваться в запросах:

```python
some_user = session.query(User).filter(User.fullname == "John Smith").first()
```

Пример конкатенации строк является простым, где выражение Python может иметь двойное назначение на уровне экземпляра и класса. Часто выражение SQL необходимо отличать от выражения Python, что можно сделать с помощью [hybrid\_property.expression()](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html#sqlalchemy.ext.hybrid.hybrid\_property.expression). Ниже мы иллюстрируем случай, когда условие должно присутствовать внутри гибрида, используя оператор **if** в Python и конструкцию [case()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.case) для выражений SQL:

```python
from sqlalchemy.ext.hybrid import hybrid_property
from sqlalchemy.sql import case

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    firstname = Column(String(50))
    lastname = Column(String(50))

    @hybrid_property
    def fullname(self):
        if self.firstname is not None:
            return self.firstname + " " + self.lastname
        else:
            return self.lastname

    @fullname.expression
    def fullname(cls):
        return case([
            (cls.firstname != None, cls.firstname + " " + cls.lastname),
        ], else_ = cls.lastname)
```

## Использование column\_property

Функцию [column\_property()](otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs) можно использовать для сопоставления SQL-выражения аналогично регулярно отображаемому столбцу [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column). При использовании этого метода атрибут загружается вместе со всеми другими атрибутами, отображаемыми столбцами, во время загрузки. В некоторых случаях это является преимуществом по сравнению с использованием гибридов, поскольку значение может быть загружено заранее одновременно с родительской строкой объекта, особенно если выражение связано с другими таблицами (обычно в виде коррелированного подзапроса). ) для доступа к данным, которые обычно недоступны для уже загруженного объекта.

Недостатки использования [column\_property()](otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs) для выражений SQL включают в себя то, что выражение должно быть совместимо с оператором **SELECT**, выдаваемым для класса в целом, а также есть некоторые особенности конфигурации, которые могут возникнуть при использовании [column\_property()](otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs) из декларативных миксинов.

Наш пример с **fullname** можно выразить с помощью [column\_property()](otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs) следующим образом:

```python
from sqlalchemy.orm import column_property

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    firstname = Column(String(50))
    lastname = Column(String(50))
    fullname = column_property(firstname + " " + lastname)
```

Также могут использоваться коррелированные подзапросы. Ниже мы используем конструкцию [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select) для создания [ScalarSelect](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.ScalarSelect), представляющего оператор **SELECT**, ориентированный на столбцы, который связывает воедино количество объектов **Address**, доступных для конкретного **User**:

```python
from sqlalchemy.orm import column_property
from sqlalchemy import select, func
from sqlalchemy import Column, Integer, String, ForeignKey

from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('user.id'))

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    address_count = column_property(
        select(func.count(Address.id)).
        where(Address.user_id==id).
        correlate_except(Address).
        scalar_subquery()
    )
```

В приведенном выше примере мы определяем конструкцию [ScalarSelect()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.ScalarSelect) следующим образом:

```python
stmt = (
    select(func.count(Address.id)).
    where(Address.user_id==id).
    correlate_except(Address).
    scalar_subquery()
)
```

Выше мы сначала используем [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select) для создания конструкции [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), которую затем преобразуем в [scalar subquery](https://docs.sqlalchemy.org/en/14/glossary.html#term-scalar-subquery) с помощью метода [Select.scalar\_subquery()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.scalar\_subquery), указывая на наше намерение использовать этот оператор [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select) в контексте выражения столбца.

В самом [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select) мы выбираем количество строк `Address.id`, где столбец `Address.user_id` приравнивается к **id**, который в контексте класса **User** является столбцом [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) с именем **id** (обратите внимание, что **id** также является именем встроенного Python). в функции, которую мы не хотим здесь использовать — если бы мы были вне определения класса **User**, мы бы использовали `User.id`).

Метод [Select.correlate\_except()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.correlate\_except) указывает, что каждый элемент в предложении **FROM** этого [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select) может быть опущен из списка **FROM** (то есть соотнесен с прилагаемым оператором **SELECT** для пользователя **User**), за исключением элемента, соответствующего **Address**. Это не является строго обязательным, но предотвращает непреднамеренное исключение **Address** из списка **FROM** в случае длинной строки соединений между таблицами **User** и **Address**, в которых вложены операторы **SELECT** для **Address**.

Если проблемы с импортом не позволяют определить свойство [column\_property()](otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs) вместе с классом, его можно назначить классу после настройки обоих. При использовании отображений, использующих базовый класс [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base), это назначение атрибута приводит к вызову [Mapper.add\_property()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.add\_property) для добавления дополнительного свойства постфактум:

```python
# работает только в том случае, если используется декларативный базовый класс
User.address_count = column_property(
    select(func.count(Address.id)).
    where(Address.user_id==User.id).
    scalar_subquery()
)
```

При использовании стилей сопоставления, которые не используют [declarative\_base()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declarative\_base), таких как декоратор [registry.mapped()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.registry.mapped), метод [Mapper.add\_property()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.add\_property) может быть вызван явно для базового объекта [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper), который можно получить с помощью [inspect()](https://docs.sqlalchemy.org/en/14/core/inspection.html#sqlalchemy.inspect):

```python
from sqlalchemy.orm import registry

reg = registry()

@reg.mapped
class User:
    __tablename__ = 'user'

    # ... дополнительные директивы сопоставления


# позже ...

# работает для любого вида отображения
from sqlalchemy import inspect
inspect(User).add_property(
    column_property(
       select(func.count(Address.id)).
       where(Address.user_id==User.id).
       scalar_subquery()
    )
)
```

Для свойства [column\_property()](otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs), которое ссылается на столбцы, связанные отношением «многие ко многим», используйте [and\_()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.and\_), чтобы соединить поля ассоциативной таблицы с обеими таблицами в отношении:

```python
from sqlalchemy import and_

class Author(Base):
    # ...

    book_count = column_property(
        select(func.count(books.c.id)
        ).where(
            and_(
                book_authors.c.author_id==authors.c.id,
                book_authors.c.book_id==books.c.id
            )
        ).scalar_subquery()
    )
```

### Составление из свойств столбца во время сопоставления

Можно создавать сопоставления, объединяющие несколько объектов [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty) вместе. [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty) будет интерпретироваться как выражение SQL при использовании в контексте основного выражения при условии, что на него нацелен существующий объект выражения; это работает благодаря тому, что ядро обнаруживает, что объект имеет метод **\_\_clause\_element\_\_()**, который возвращает выражение SQL. Однако, если свойство [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty) используется в качестве ведущего объекта в выражении, где нет другого объекта выражения Core SQL для его назначения, атрибут [ColumnProperty.expression](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty.expression) вернет базовое выражение SQL, чтобы его можно было использовать для согласованного построения выражений SQL. Ниже класс **File** содержит атрибут `File.path`, который объединяет токен строки с атрибутом `File.filename`, который сам является [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty):

```python
class File(Base):
    __tablename__ = 'file'

    id = Column(Integer, primary_key=True)
    name = Column(String(64))
    extension = Column(String(8))
    filename = column_property(name + '.' + extension)
    path = column_property('C:/' + filename.expression)
```

Когда класс **File** обычно используется в выражениях, атрибуты, присвоенные **filename** и **path**, можно использовать напрямую. Использование атрибута [ColumnProperty.expression](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty.expression) необходимо только при использовании [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty) непосредственно в определении сопоставления:

```python
q = session.query(File.path).filter(File.filename == 'foo.txt')
```

## Использование простого дескриптора

В тех случаях, когда необходимо сгенерировать SQL-запрос, более сложный, чем то, что может обеспечить [column\_property()](otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs) или [hybrid\_property](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html#sqlalchemy.ext.hybrid.hybrid\_property), можно использовать обычную функцию Python, доступ к которой осуществляется как атрибут, предполагая, что выражение должно быть доступно только в уже загруженном экземпляре. Функция украшена собственным декоратором Python `@property`, чтобы пометить ее как атрибут только для чтения. Внутри функции [object\_session()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.object\_session) используется для поиска сеанса [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), соответствующего текущему объекту, который затем используется для отправки запроса:

```python
from sqlalchemy.orm import object_session
from sqlalchemy import select, func

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    firstname = Column(String(50))
    lastname = Column(String(50))

    @property
    def address_count(self):
        return object_session(self).\
            scalar(
                select(func.count(Address.id)).\
                    where(Address.user_id==self.id)
            )
```

Подход с простым дескриптором полезен в крайнем случае, но в обычном случае он менее эффективен, чем гибридный подход и подход со свойствами столбца, поскольку он должен выдавать SQL-запрос при каждом доступе.

## SQL-выражения во время запроса как сопоставленные атрибуты

При использовании [Session.query()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.query) у нас есть возможность указать не только сопоставленные объекты, но и специальные выражения SQL. Предположим, что если бы класс **A** имел целочисленные атрибуты `.x` и `.y`, мы могли бы запросить объекты **A** и дополнительно сумму `.x` и `.y` следующим образом:

```python
q = session.query(A, A.x + A.y)
```

Приведенный выше запрос возвращает кортежи формы `(A object, integer)`.

Существует опция, которая может применять специальное выражение `A.x + A.y` к возвращенным объектам **A** вместо отдельной записи кортежа; это параметр запроса [with\_expression()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.with\_expression) в сочетании с сопоставлением атрибутов [query\_expression()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.query\_expression). Класс сопоставляется с атрибутом-заполнителем, к которому можно применить любое конкретное выражение SQL:

```python
from sqlalchemy.orm import query_expression

class A(Base):
    __tablename__ = 'a'
    id = Column(Integer, primary_key=True)
    x = Column(Integer)
    y = Column(Integer)

    expr = query_expression()
```

Затем мы можем запросить объекты типа **A**, применяя произвольное выражение SQL для заполнения в `A.expr`:

```python
from sqlalchemy.orm import with_expression
q = session.query(A).options(
    with_expression(A.expr, A.x + A.y))
```

Сопоставление [query\_expression()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.query\_expression) имеет следующие оговорки:

* В объекте, где [query\_expression()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.query\_expression) не использовался для заполнения атрибута, атрибут экземпляра объекта будет иметь значение `None`, если для параметра [query\_expression.default\_expr](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.query\_expression.params.default\_expr) не задано альтернативное выражение SQL.
* Значение `query_expression` **не заполняется для уже загруженного объекта**. То есть так **не получится**:

```python
obj = session.query(A).first()

obj = session.query(A).options(with_expression(A.expr, some_expr)).first()
```

Чтобы обеспечить повторную загрузку атрибута, используйте [Query.populate\_existing()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.populate\_existing):

```python
obj = session.query(A).populate_existing().options(
    with_expression(A.expr, some_expr)).first()
```

* Значение `query_expression` **не обновляется, когда срок действия объекта истек**. Как только срок действия объекта истекает либо через [Session.expire()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.expire), либо через поведение `expire_on_commit` [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit), значение удаляется из атрибута и при последующем доступе возвращается `None`. Только при запуске нового запроса, который касается объекта, содержащего новую директиву [with\_expression()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.with\_expression), для атрибута будет установлено значение, отличное от `None`.
* Сопоставленный атрибут в настоящее время нельзя применить к другим частям запроса, таким как предложение **WHERE**, предложение **ORDER BY**, и использовать специальное выражение; то есть это не сработает:

```python
# не работает
q = session.query(A).options(
    with_expression(A.expr, A.x + A.y)
).filter(A.expr > 5).order_by(A.expr)
```

Выражение `A.expr` разрешается в **NULL** в приведенном выше предложении **WHERE** и предложении **ORDER BY**. Чтобы использовать выражение во всем запросе, назначьте его переменной и используйте ее:

```python
a_expr = A.x + A.y
q = session.query(A).options(
    with_expression(A.expr, a_expr)
).filter(a_expr > 5).order_by(a_expr)
```

{% hint style="warning" %}
**New in version 1.2.**
{% endhint %}
