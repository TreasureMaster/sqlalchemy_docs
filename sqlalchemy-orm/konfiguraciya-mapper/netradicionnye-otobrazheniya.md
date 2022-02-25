# Нетрадиционные отображения

## Нетрадиционные отображения

### Сопоставление класса с несколькими таблицами

Mappers могут быть созданы для произвольных реляционных единиц (называемых _**selectables**_) в дополнение к простым таблицам. Например, функция [join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.join) создает выбираемую единицу, состоящую из нескольких таблиц, дополненную собственным составным первичным ключом, который можно сопоставить так же, как таблицу [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table):

```python
from sqlalchemy import Table, Column, Integer, \
        String, MetaData, join, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import column_property

metadata_obj = MetaData()

# определить 2 объекта Table
user_table = Table('user', metadata_obj,
            Column('id', Integer, primary_key=True),
            Column('name', String),
        )

address_table = Table('address', metadata_obj,
            Column('id', Integer, primary_key=True),
            Column('user_id', Integer, ForeignKey('user.id')),
            Column('email_address', String)
            )

# определить объединение между ними.
# Это происходит в столбцах user.id и address.user_id.
user_address_join = join(user_table, address_table)

Base = declarative_base()

# сопоставить с ним
class AddressUser(Base):
    __table__ = user_address_join

    id = column_property(user_table.c.id, address_table.c.user_id)
    address_id = address_table.c.id
```

В приведенном выше примере объединение выражает столбцы как для **user**, так и для таблицы **address**. Столбцы **user.id** и **address.user\_id** приравниваются внешним ключом, поэтому при сопоставлении они определяются как один атрибут, **AddressUser.id**, с помощью [column\_property()](otobrazhenie-kolonok-i-vyrazhenii/otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs) для указания специализированного сопоставления столбцов. На основе этой части конфигурации сопоставление скопирует новые значения первичного ключа из **user.id** в столбец **address.user\_id**, когда произойдет сброс flush.

Кроме того, столбец **address.id** явно сопоставляется с атрибутом с именем **address\_id**. Это необходимо **для устранения неоднозначности** сопоставления столбца **address.id** с одноименным атрибутом **AddressUser.id**, который здесь был назначен для ссылки на таблицу пользователей в сочетании с внешним ключом **address.user\_id**.

Естественный первичный ключ приведенного выше сопоставления является составным из `(user.id, address.id)`, поскольку они представляют собой объединенные вместе столбцы первичного ключа таблицы **user** и **address**. Идентификатор объекта **AddressUser** будет основываться на этих двух значениях и представлен из объекта **AddressUser** как `(AddressUser.id, AddressUser.address_id)`.

При ссылке на столбец **AddressUser.id** в большинстве выражений SQL будет использоваться только первый столбец в списке сопоставленных столбцов, поскольку эти два столбца являются синонимами. Однако для особого случая использования, такого как выражение **GROUP BY**, когда на оба столбца необходимо ссылаться одновременно, используя надлежащий контекст, то есть приспосабливаясь к псевдонимам и т. п., можно использовать метод доступа [Comparator.expressions](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty.Comparator.expressions):

```python
q = session.query(AddressUser).group_by(*AddressUser.id.expressions)
```

{% hint style="warning" %}
**Новое в версии 1.3.17**: добавлен метод доступа [Comparator.expressions](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty.Comparator.expressions).
{% endhint %}

{% hint style="info" %}
Сопоставление с несколькими таблицами, как показано выше, поддерживает сохраняемость, то есть **INSERT**, **UPDATE** и **DELETE** строк в целевых таблицах. Однако он не поддерживает операцию, которая **UPDATE** одну таблицу и выполняет **INSERT** или **DELETE** в других одновременно для одной записи. То есть, если запись `PtoQ` сопоставляется с таблицами `«p»` и `«q»`, где она имеет строку, основанную на **LEFT OUTER JOIN** `«p»` и `«q»`, если выполняется **UPDATE**, которое должно изменить данные в таблица `«q»` в существующей записи, строка в `«q»` должна существовать; он не выдаст **INSERT**, если идентификатор первичного ключа уже присутствует. Если строка не существует, для большинства драйверов DBAPI, поддерживающих отчет о количестве строк, затронутых **UPDATE**, ORM не сможет обнаружить обновленную строку и выдаст ошибку; в противном случае данные будут молча игнорироваться.

Рецепт, позволяющий «вставить» связанную строку на лету, может использовать событие `.MapperEvents.before_update` и выглядеть так:

```python
from sqlalchemy import event

@event.listens_for(PtoQ, 'before_update')
def receive_before_update(mapper, connection, target):
   if target.some_required_attr_on_q is None:
        connection.execute(q_table.insert(), {"id": target.id})
```

где выше строка **INSERT** в таблицу **q\_table** путем создания конструкции **INSERT** с помощью [Table.insert()](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table.insert), а затем ее выполнения с использованием данного соединения [Connection](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Connection), которое является тем же самым, которое используется для выдачи другого SQL для процесса очистки. Пользовательская логика должна обнаружить, что **LEFT OUTER JOIN** от `«p»` до `«q»` не имеет записи для стороны «q».
{% endhint %}

### Сопоставление класса с произвольными подзапросами

Подобно сопоставлению с объединением, простой объект [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select) также может использоваться с преобразователем mapper. Фрагмент примера ниже иллюстрирует сопоставление класса **Customer** с [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), который включает соединение с подзапросом:

```python
from sqlalchemy import select, func

subq = select(
    func.count(orders.c.id).label('order_count'),
    func.max(orders.c.price).label('highest_order'),
    orders.c.customer_id
).group_by(orders.c.customer_id).subquery()

customer_select = select(customers, subq).join_from(
    customers, subq, customers.c.id == subq.c.customer_id
).subquery()

class Customer(Base):
    __table__ = customer_select
```

Выше полная строка, представленная **customer\_select**, будет состоять из всех столбцов таблицы **customers** в дополнение к столбцам, предоставляемым подзапросом **subq**, а именно: **order\_count**, **highest\_order** и **customer\_id**. Сопоставление класса **Customer** с этим выбираемым затем создает класс, который будет содержать эти атрибуты.

Когда ORM сохраняет новые экземпляры **Customer**, только таблица **customers** получит **INSERT**. Это связано с тем, что первичный ключ таблицы **orders** не представлен в отображении; ORM будет выдавать **INSERT** только в таблицу, для которой он сопоставил первичный ключ.

{% hint style="info" %}
Практика сопоставления с произвольными операторами **SELECT**, особенно сложными, как указано выше, почти никогда не требуется; он обязательно имеет тенденцию создавать сложные запросы, которые часто менее эффективны, чем те, которые были бы получены при прямом построении запроса. Практика в некоторой степени основана на очень ранней истории SQLAlchemy, где конструкция [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper) предназначалась для представления основного интерфейса запросов; в современном использовании объект [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) можно использовать для создания практически любого оператора **SELECT**, включая сложные составные объекты, и ему следует отдавать предпочтение перед подходом «отображать к выбираемому».
{% endhint %}

### Несколько Mappers для одного класса

В современной SQLAlchemy конкретный класс отображается только одним так называемым **первичным** преобразователем одновременно. Этот преобразователь задействован в трех основных областях функциональности: запросы, сохраняемость и инструментирование сопоставленного класса. Обоснование основного преобразователя связано с тем фактом, что [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper) изменяет сам класс, не только сохраняя его в отношении конкретной таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), но и **настраивая** атрибуты класса, которые структурированы специально в соответствии с метаданными таблицы. Невозможно, чтобы более одного преобразователя были связаны с классом в равной мере, поскольку только один преобразователь может фактически инструментировать класс.

Концепция **«non-primary»** преобразователя существовала во многих версиях SQLAlchemy, однако начиная с версии 1.3 эта функция устарела. Единственный случай, когда такой неосновной преобразователь полезен, — это построение отношения к классу против альтернативного выбираемого. Этот вариант использования теперь подходит для использования конструкции с **aliased** и описан в разделе Отношение к классу с псевдонимом ([Relationship to Aliased Class](https://docs.sqlalchemy.org/en/14/orm/join\_conditions.html#relationship-aliased-class)).

Что касается варианта использования класса, который фактически может быть полностью сохранен в разных таблицах в разных сценариях, очень ранние версии SQLAlchemy предлагали для этого функцию, адаптированную из **Hibernate**, известную как функция **«entity name»**. Однако этот вариант использования стал невозможным в SQLAlchemy, как только сопоставленный класс сам стал источником построения выражения SQL; то есть сами атрибуты класса напрямую связаны со столбцами отображаемой таблицы. Эта функция была удалена и заменена простым ориентированным на рецепт подходом к выполнению этой задачи без какой-либо двусмысленности инструментовки - для создания новых подклассов, каждый из которых отображается индивидуально. Этот шаблон теперь доступен в виде рецепта на [Entity Name](https://www.sqlalchemy.org/trac/wiki/UsageRecipes/EntityName).
