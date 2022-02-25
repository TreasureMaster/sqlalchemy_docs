# Работа со связанными объектами

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Эта страница является частью [учебника по SQLAlchemy 1.4/2.0](./).

Предыдущий: [Манипуляции с данными с помощью ORM](manipulyacii-s-dannymi-s-pomoshyu-orm.md) | Далее: [Дополнительная литература](dopolnitelnoe-chtenie.md)
{% endhint %}

В этом разделе мы рассмотрим еще одну важную концепцию ORM, а именно то, как ORM взаимодействует с сопоставленными классами, которые ссылаются на другие объекты. В разделе «[Объявление сопоставленных классов](rabota-s-metadannymi-bazy-dannykh.md#obyavlenie-sopostavlennykh-mapped-klassov)» в примерах сопоставленных классов использовалась конструкция, называемая отношением [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for). Эта конструкция определяет связь между двумя разными сопоставленными классами или между сопоставленным классом с самим собой, последнее из которых называется **самореферентным отношением**.

Чтобы описать основную идею отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), сначала мы рассмотрим сопоставление в краткой форме, опуская сопоставления столбцов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) и другие директивы:

```python
from sqlalchemy.orm import relationship

class User(Base):
    __tablename__ = 'user_account'

    # ... Column mappings

    addresses = relationship("Address", back_populates="user")


class Address(Base):
    __tablename__ = 'address'

    # ... Column mappings

    user = relationship("User", back_populates="addresses")
```

Выше класс **User** теперь имеет атрибут **User.addresses**, а класс **Address** имеет атрибут **Address.user**. Конструкция [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) будет использоваться для проверки взаимосвязей таблиц между объектами [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), которые сопоставлены с классами **User** и **Address**. Поскольку объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), представляющий таблицу адресов **address**, имеет [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint), который ссылается на таблицу **user\_account**, [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) может однозначно определить, что имеется отношение **один ко многим** ([one to many](https://docs.sqlalchemy.org/en/14/glossary.html#term-one-to-many)) между **User.addresses** и **User**; на одну конкретную строку в таблице **user\_account** может ссылаться множество строк в таблице адресов **address**.

Все отношения «**один ко многим**» естественным образом соответствуют отношениям «**многие к одному**» ([many to one](https://docs.sqlalchemy.org/en/14/glossary.html#term-many-to-one)) в другом направлении, в данном случае тому, которое было отмечено **Address.user**. Параметр [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates), показанный выше, настроенный для обоих объектов [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), ссылающихся на другое имя, устанавливает, что каждая из этих двух конструкций [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) должна рассматриваться как дополняющая друг друга; мы увидим, как это происходит в следующем разделе.

## Сохранение и загрузка отношений (relationships)

Мы можем начать с иллюстрации того, что делает отношение [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) с экземплярами объектов. Если мы создадим новый объект **User**, мы можем заметить, что при доступе к элементу `.addresses` есть список Python:

```python
>>> u1 = User(name='pkrabs', fullname='Pearl Krabs')
>>> u1.addresses
[]
```

Этот объект представляет собой специфичную для SQLAlchemy версию списка **list** Python, которая имеет возможность отслеживать внесенные в нее изменения и реагировать на них. Коллекция также появлялась автоматически, когда мы обращались к атрибуту, хотя мы никогда не присваивали его объекту. Это похоже на поведение, отмеченное при [вставке строк с помощью ORM](manipulyacii-s-dannymi-s-pomoshyu-orm.md#vstavka-strok-s-orm), где было замечено, что атрибуты на основе столбцов, которым мы явно не присваиваем значение, также автоматически отображаются как `None`, а не вызывают **AttributeError**, как обычное поведение Python.

Поскольку объект **u1** все еще является временным ([transient](https://docs.sqlalchemy.org/en/14/glossary.html#term-transient)), а список **list**, который мы получили из **u1.addresses**, не был изменен (т.е. добавлен или расширен), он еще не связан с объектом, но когда мы внесем в него изменения, он станет частью состояния объекта **User**.

Коллекция специфична для класса **Address**, который является единственным типом объекта Python, который может быть сохранен в ней. Используя метод `list.append()`, мы можем добавить объект **Address**:

```python
>>> a1 = Address(email_address="pearl.krabs@gmail.com")
>>> u1.addresses.append(a1)
```

На данный момент коллекция **u1.addresses**, как и ожидалось, содержит новый объект **Address**:

```python
>>> u1.addresses
[Address(id=None, email_address='pearl.krabs@gmail.com')]
```

Поскольку мы связали объект **Address** с коллекцией **User.addresses** экземпляра **u1**, также произошло другое поведение, заключающееся в том, что отношение **User.addresses** синхронизировалось с отношением **Address.user**, так что мы можем перемещаться не только из объекта **User** к объекту **Address**, мы также можем перейти от объекта **Address** обратно к «родительскому» объекту **User**:

```python
>>> a1.user
User(id=None, name='pkrabs', fullname='Pearl Krabs')
```

_**Эта синхронизация произошла в результате использования нами параметра**_ [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) между двумя объектами отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for). Этот параметр называет другое отношение [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), для которого должно произойти дополнительное присвоение атрибута/изменение списка. Это будет одинаково хорошо работать и в другом направлении, а именно в том, что если мы создадим другой объект **Address** и назначим пользователя его атрибуту **Address.user**, этот **Address** станет частью коллекции **User.addresses** для этого объекта **User**:

```python
>>> a2 = Address(email_address="pearl@aol.com", user=u1)
>>> u1.addresses
[Address(id=None, email_address='pearl.krabs@gmail.com'),
 Address(id=None, email_address='pearl@aol.com')]
```

На самом деле мы использовали параметр **user** в качестве аргумента ключевого слова в конструкторе **Address**, который принимается так же, как и любой другой сопоставленный атрибут, объявленный в классе **Address**. Это эквивалентно присвоению атрибута **Address.user** постфактум:

```python
# эквивалентно эффекту a2 = Address(user=u1)
>>> a2.user = u1
```

### Каскадирование объектов в сеансе

Теперь у нас есть объект **User** и два объекта **Address**, которые связаны двунаправленной структурой в памяти, но, как отмечалось ранее в разделе «[Вставка строк с помощью ORM](manipulyacii-s-dannymi-s-pomoshyu-orm.md#vstavka-strok-s-orm)», эти объекты считаются находящимися в переходном ([transient](https://docs.sqlalchemy.org/en/14/glossary.html#term-transient)) состоянии до тех пор, пока они не будут связаны с объектом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session).

Мы используем сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), который все еще продолжается, и обратите внимание, что когда мы применяем метод [Session.add()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.add) к ведущему объекту пользователя **User**, связанный объект **Address** также добавляется к тому же сеансу [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session):

```python
>>> session.add(u1)
>>> u1 in session
True
>>> a1 in session
True
>>> a2 in session
True
```

Вышеупомянутое поведение, когда сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) получил объект пользователя **User** и следовал отношению **User.addresses**, чтобы найти связанный объект **Address**, известен как **каскад save-update** и подробно обсуждается в справочной документации ORM на Cascades ([Cascades](https://docs.sqlalchemy.org/en/14/orm/cascades.html#unitofwork-cascades)).

Три объекта теперь находятся в состоянии ожидания ([pending](https://docs.sqlalchemy.org/en/14/glossary.html#term-pending)); это означает, что они готовы стать предметом операции **INSERT**, но это еще не произошло; всем трем объектам еще не назначен первичный ключ, и, кроме того, объекты **a1** и **a2** имеют атрибут с именем **user\_id**, который ссылается на столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), имеющий [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint), ссылающийся на столбец **user\_account.id**; который также равен `None`, поскольку объекты еще не связаны с реальной строкой базы данных:

```python
>>> print(u1.id)
None
>>> print(a1.user_id)
None
```

Именно на этом этапе мы можем увидеть очень большую полезность, которую обеспечивает единица рабочего процесса; напомним, что в разделе [INSERT обычно автоматически генерируется предложение «values»](rabota-s-dannymi/vstavka-strok-s-core.md#insert-obychno-avtomaticheski-generiruet-predlozhenie-values), строки были вставлены в таблицы **user\_account** и **address** с использованием сложного синтаксиса для автоматического связывания столбцов **address.user\_id** со столбцами в строках **user\_account**. Кроме того, было необходимо, чтобы мы сначала выдавали **INSERT** для строк **user\_account**, а затем для адресов **address**, поскольку строки в **address** _**зависят**_ от их родительской строки в **user\_account** для значения в их столбце **user\_id**.

При использовании сеанса [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) вся эта утомительная работа выполняется за нас, и даже самые стойкие сторонники чистоты SQL могут извлечь выгоду из автоматизации операторов **INSERT**, **UPDATE** и **DELETE**. Когда мы фиксируем [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit) транзакцию, все шаги вызываются в правильном порядке, и, кроме того, вновь сгенерированный первичный ключ строки **user\_account** применяется к столбцу **address.user\_id** соответствующим образом:

```python
>>> session.commit()
```

```sql
INSERT INTO user_account (name, fullname) VALUES (?, ?)
[...] ('pkrabs', 'Pearl Krabs')
INSERT INTO address (email_address, user_id) VALUES (?, ?)
[...] ('pearl.krabs@gmail.com', 6)
INSERT INTO address (email_address, user_id) VALUES (?, ?)
[...] ('pearl@aol.com', 6)
COMMIT
```

## Загрузка отношений (relationships)

На последнем шаге мы вызвали [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit), который выдал **COMMIT** для транзакции, а затем для [Session.commit.expire\_on\_commit](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit.params.expire\_on\_commit) _**истек срок действия всех объектов**_, чтобы они обновлялись для следующей транзакции.

Когда мы в следующий раз получим доступ к атрибуту этих объектов, мы увидим **SELECT**, испускаемый для первичных атрибутов строки, например, когда мы просматриваем только что сгенерированный первичный ключ для объекта **u1**:

```python
>>> u1.id
```

```sql
BEGIN (implicit)
SELECT user_account.id AS user_account_id, user_account.name AS user_account_name,
user_account.fullname AS user_account_fullname
FROM user_account
WHERE user_account.id = ?
[...] (6,)
```

```bash
6
```

Объект **u1 User** теперь имеет постоянную коллекцию **User.addresses**, к которой мы также можем получить доступ. Поскольку эта коллекция состоит из дополнительного набора строк из таблицы адресов **address**, когда мы также обращаемся к этой коллекции, мы снова видим ленивую загрузку ([lazy load](https://docs.sqlalchemy.org/en/14/glossary.html#term-lazy-load)), выполняемую для извлечения объектов:

```python
>>> u1.addresses
```

```sql
SELECT address.id AS address_id, address.email_address AS address_email_address,
address.user_id AS address_user_id
FROM address
WHERE ? = address.user_id
[...] (6,)
```

```python
[Address(id=4, email_address='pearl.krabs@gmail.com'),
 Address(id=5, email_address='pearl@aol.com')]
```

Коллекции и связанные атрибуты в ORM SQLAlchemy сохраняются в памяти; после заполнения коллекции или атрибута SQL больше не выдается до тех пор, пока не истечет срок действия ([expired](https://docs.sqlalchemy.org/en/14/glossary.html#term-expired)) этой коллекции или атрибута. Мы можем снова получить доступ к **u1.addresses**, а также добавить или удалить элементы, и это не повлечет за собой никаких новых вызовов SQL:

```python
>>> u1.addresses
[Address(id=4, email_address='pearl.krabs@gmail.com'),
 Address(id=5, email_address='pearl@aol.com')]
```

В то время как нагрузка, создаваемая отложенной загрузкой, может быстро стать дорогостоящей, если мы не предпримем явных шагов для ее оптимизации, сеть отложенной загрузки, по крайней мере, достаточно хорошо оптимизирована, чтобы не выполнять избыточную работу; поскольку коллекция **u1.addresses** была обновлена, согласно карте идентификации ([identity map](https://docs.sqlalchemy.org/en/14/glossary.html#term-identity-map)) это фактически те же экземпляры **Address**, такие как объекты **a1** и **a2**, с которыми мы уже имели дело, поэтому мы закончили загрузку всех атрибутов в этом конкретном графе объектов:

```python
>>> a1
Address(id=4, email_address='pearl.krabs@gmail.com')
>>> a2
Address(id=5, email_address='pearl@aol.com')
```

Вопрос о том, насколько отношения загружаются или нет, является отдельной темой. Некоторое дополнительное введение в эти концепции приведено ниже в этом разделе, в разделе «[Стратегии загрузчика](rabota-so-svyazannymi-obektami.md#strategii-zagruzki)».

## Использование отношений (relationships) в запросах (Query)

В предыдущем разделе было представлено поведение конструкции [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) при работе с **экземплярами отображаемого класса**, выше — с экземплярами **u1**, **a1** и **a2** классов **User** и **Address**. В этом разделе мы познакомим вас с поведением отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), поскольку оно применимо к **поведению отображаемого класса на уровне класса**, где оно служит несколькими способами, помогающими автоматизировать построение SQL-запросов.

### Использование отношений (relationship) в JOIN

В разделах [Явные предложения FROM и JOIN](rabota-s-dannymi/vybor-strok-s-pomoshyu-core-ili-orm.md#yavnye-usloviya-from-i-join), а также [Настройка предложения ON](rabota-s-dannymi/vybor-strok-s-pomoshyu-core-ili-orm.md#ustanovka-predlozheniya-on) описано использование методов [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join) и [Select.join\_from()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join\_from) для составления предложений SQL **JOIN**. Чтобы описать, как соединить таблицы, эти методы либо **выводят** предложение **ON** на основе наличия единственного однозначного объекта [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint) в структуре метаданных таблицы, которая связывает две таблицы, либо в противном случае мы можем предоставить явную конструкцию выражения SQL, которая указывает конкретное предложение **ON**.

При использовании сущностей ORM доступен дополнительный механизм, помогающий нам настроить предложение **ON** соединения, которое заключается в использовании объектов отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), которые мы настроили в нашем пользовательском сопоставлении, как было продемонстрировано в разделе «[Объявление сопоставленных классов](rabota-s-metadannymi-bazy-dannykh.md#obyavlenie-sopostavlennykh-mapped-klassov)». Привязанный к классу атрибут, соответствующий отношениям [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), может быть передан в качестве **единственного аргумента** в [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join), где он служит для одновременного указания как правой стороны соединения, так и предложения **ON**:

```python
>>> print(
...     select(Address.email_address).
...     select_from(User).
...     join(User.addresses)
... )
```

```sql
SELECT address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
```

Наличие отношения ORM [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) в отображении _mapping_ не используется функциями [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join) или [Select.join\_from()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join\_from), если мы его не указываем; он не используется для вывода предложения **ON**. Это означает, что если мы присоединяемся от **User** к **Address** без предложения **ON**, это работает из-за [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint) между двумя сопоставленными объектами [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), а не из-за объектов отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) в классах **User** и **Address**:

```python
>>> print(
...    select(Address.email_address).
...    join_from(User, Address)
... )
```

```sql
SELECT address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
```

### Объединение между целями с псевдонимами

В разделе «[Псевдонимы объекта ORM](rabota-s-dannymi/vybor-strok-s-pomoshyu-core-ili-orm.md#psevdonimy-obektov-orm)» мы представили конструкцию [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased), которая используется для применения псевдонима SQL к объекту ORM. При использовании отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) для создания SQL **JOIN** вариант использования, когда целью соединения должен быть псевдоним [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased), подходит использование модификатора [PropComparator.of\_type()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator.of\_type). Чтобы продемонстрировать, мы создадим такое же соединение, как показано в [ORM Entity Aliases](rabota-s-dannymi/vybor-strok-s-pomoshyu-core-ili-orm.md#psevdonimy-obektov-orm), используя вместо этого атрибуты [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) для соединения:

```python
>>> print(
...        select(User).
...        join(User.addresses.of_type(address_alias_1)).
...        where(address_alias_1.email_address == 'patrick@aol.com').
...        join(User.addresses.of_type(address_alias_2)).
...        where(address_alias_2.email_address == 'patrick@gmail.com')
...    )
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
JOIN address AS address_1 ON user_account.id = address_1.user_id
JOIN address AS address_2 ON user_account.id = address_2.user_id
WHERE address_1.email_address = :email_address_1
AND address_2.email_address = :email_address_2
```

Чтобы использовать связь [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) для создания соединения **из** объекта с псевдонимом, атрибут доступен непосредственно из конструкции [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased):

```python
>>> user_alias_1 = aliased(User)
>>> print(
...     select(user_alias_1.name).
...     join(user_alias_1.addresses)
... )
```

```sql
SELECT user_account_1.name
FROM user_account AS user_account_1
JOIN address ON user_account_1.id = address.user_id
```

### Расширение критериев ON

Предложение **ON**, сгенерированное конструкцией [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), также может быть дополнено дополнительными критериями. Это полезно как для быстрых способов ограничить область действия конкретного объединения по пути связи, так и для вариантов использования, таких как настройка стратегий загрузчика, представленных ниже в разделе [Стратегии загрузчика](rabota-so-svyazannymi-obektami.md#strategii-zagruzki). Метод [PropComparator.and\_()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator.and\_) позиционно принимает серию выражений SQL, которые будут присоединены к предложению **ON** оператора **JOIN** через **AND**. Например, если мы хотим **JOIN** от пользователя **User** к адресу **Address**, но также ограничим критерии включения только определенными адресами электронной почты:

```python
>>> stmt = (
...   select(User.fullname).
...   join(User.addresses.and_(Address.email_address == 'pearl.krabs@gmail.com'))
... )
>>> session.execute(stmt).all()
```

```sql
SELECT user_account.fullname
FROM user_account
JOIN address ON user_account.id = address.user_id AND address.email_address = ?
[...] ('pearl.krabs@gmail.com',)
```

```python
[('Pearl Krabs',)]
```

### Формы EXISTS: has() / any()

В разделе [подзапросы EXISTS](rabota-s-dannymi/vybor-strok-s-pomoshyu-core-ili-orm.md#podzaprosy-exists) мы представили объект [Exists](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Exists), который предоставляет ключевое слово SQL **EXISTS** в сочетании со скалярным подзапросом. Конструкция отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) предоставляет некоторые вспомогательные методы, которые можно использовать для генерации некоторых общих стилей запросов **EXISTS** с точки зрения отношения.

Для отношения «**один ко многим**», такого как **User.addresses**, **EXISTS** для таблицы адресов **address**, которая коррелирует обратно с таблицей **user\_account**, может быть создана с помощью [PropComparator.any()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator.any). Этот метод принимает необязательный критерий **WHERE** для ограничения строк, соответствующих подзапросу:

```python
>>> stmt = (
...   select(User.fullname).
...   where(User.addresses.any(Address.email_address == 'pearl.krabs@gmail.com'))
... )
>>> session.execute(stmt).all()
```

```sql
SELECT user_account.fullname
FROM user_account
WHERE EXISTS (SELECT 1
FROM address
WHERE user_account.id = address.user_id AND address.email_address = ?)
[...] ('pearl.krabs@gmail.com',)
```

```bash
[('Pearl Krabs',)]
```

Поскольку **EXISTS** имеет тенденцию быть более эффективным для отрицательного поиска, общий запрос состоит в том, чтобы найти сущности, где нет связанных сущностей. Это лаконично, если использовать такую фразу, как `~User.addresses.any()`, для выбора объектов **User**, которые не имеют связанных адресных строк **Address**:

```python
>>> stmt = (
...   select(User.fullname).
...   where(~User.addresses.any())
... )
>>> session.execute(stmt).all()
```

```sql
SELECT user_account.fullname
FROM user_account
WHERE NOT (EXISTS (SELECT 1
FROM address
WHERE user_account.id = address.user_id))
[...] ()
```

```bash
[('Patrick McStar',), ('Squidward Tentacles',), ('Eugene H. Krabs',)]
```

Метод [PropComparator.has()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator.has) работает в основном так же, как [PropComparator.any()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator.any), за исключением того, что он используется для отношений «**многие к одному**», например, если мы хотим найти все объекты **Address**, принадлежащие «pearl»:

```python
>>> stmt = (
...   select(Address.email_address).
...   where(Address.user.has(User.name=="pkrabs"))
... )
>>> session.execute(stmt).all()
```

```sql
SELECT address.email_address
FROM address
WHERE EXISTS (SELECT 1
FROM user_account
WHERE user_account.id = address.user_id AND user_account.name = ?)
[...] ('pkrabs',)
```

```bash
[('pearl.krabs@gmail.com',), ('pearl@aol.com',)]
```

### Общие операторы отношений (relationship)

Есть несколько дополнительных разновидностей помощников генерации SQL, которые поставляются с отношениями [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), в том числе:

* **сравнение равно отношения «многие к одному»** — конкретный экземпляр объекта можно сравнить с отношением «**многие к одному**», чтобы выбрать строки, в которых внешний ключ целевого объекта соответствует значению первичного ключа данного объекта:

```python
>>> print(select(Address).where(Address.user == u1))
```

```sql
SELECT address.id, address.email_address, address.user_id
FROM address
WHERE :param_1 = address.user_id
```

* **сравнение не равно отношения «многие к одному»** — также можно использовать оператор «**не равно**»:

```python
>>> print(select(Address).where(Address.user != u1))
```

```sql
SELECT address.id, address.email_address, address.user_id
FROM address
WHERE address.user_id != :user_id_1 OR address.user_id IS NULL
```

* **объект содержится в коллекции «один ко многим»** — это, по сути, версия сравнения «**равно**» «**один ко многим**», выбирает строки, в которых первичный ключ равен значению внешнего ключа в связанном объекте:

```python
>>> print(select(User).where(User.addresses.contains(a1)))
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.id = :param_1
```

* **объект имеет определенного родителя с точки зрения «один ко многим»** — функция [with\_parent()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.with\_parent) производит сравнение, которое возвращает строки, на которые ссылается данный родитель, это по сути то же самое, что и использование оператора `==` со стороны многие-к-одному:

```python
>>> from sqlalchemy.orm import with_parent
>>> print(select(Address).where(with_parent(u1, User.addresses)))
```

```sql
SELECT address.id, address.email_address, address.user_id
FROM address
WHERE :param_1 = address.user_id
```

## Стратегии загрузчика

В разделе «[Загрузка отношений](rabota-so-svyazannymi-obektami.md#zagruzka-otnoshenii-relationships)» мы представили концепцию, согласно которой, когда мы работаем с экземплярами сопоставленных объектов, доступ к атрибутам, которые сопоставлены с помощью отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) в случае по умолчанию, вызовет ленивую загрузку ([lazy load](https://docs.sqlalchemy.org/en/14/glossary.html#term-lazy-load)), когда коллекция не заполнена для загрузки объектов, которые должны быть в этой коллекции.

**Ленивая загрузка** — один из самых известных шаблонов ORM, а также наиболее противоречивый. Когда каждый из нескольких десятков ORM-объектов в памяти ссылается на несколько незагруженных атрибутов, рутинные манипуляции с этими объектами могут повлечь за собой множество дополнительных запросов, которые могут складываться (также известные как проблема N плюс один ([N plus one problem](https://docs.sqlalchemy.org/en/14/glossary.html#term-N-plus-one-problem))), и, что еще хуже, они генерируются **неявно**. Эти неявные запросы могут быть незаметны, могут вызывать ошибки при попытке их выполнения после того, как транзакция базы данных больше не доступна, или при использовании альтернативных шаблонов параллелизма, таких как [asyncio](https://docs.sqlalchemy.org/en/14/orm/extensions/asyncio.html), они **вообще не будут работать**.

В то же время отложенная загрузка является чрезвычайно популярным и полезным шаблоном, если он совместим с используемым подходом параллелизма и не вызывает проблем в других отношениях. По этим причинам ORM SQLAlchemy уделяет большое внимание возможности контролировать и оптимизировать такое поведение загрузки.

Прежде всего, первым шагом в эффективном использовании ленивой загрузки ORM является **тестирование приложения, включение эха SQL и просмотр выдаваемого SQL**. Если кажется, что есть много избыточных операторов **SELECT**, которые очень похожи на то, что их можно было бы объединить в один гораздо более эффективно, если есть неуместные загрузки для объектов, которые были отсоединены ([detached](https://docs.sqlalchemy.org/en/14/glossary.html#term-detached)) от их сеанса [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), тогда следует изучить **стратегии загрузчика**.

Стратегии загрузчика представлены в виде объектов, которые могут быть связаны с оператором **SELECT** с помощью метода [Select.options()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.options), например:

```python
for user_obj in session.execute(
    select(User).options(selectinload(User.addresses))
).scalars():
    user_obj.addresses  # доступная коллекция адресов уже загружена
```

Они также могут быть настроены как значения по умолчанию для отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) с использованием параметра [relationship.lazy](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.lazy), например:

```python
from sqlalchemy.orm import relationship
class User(Base):
    __tablename__ = 'user_account'

    addresses = relationship("Address", back_populates="user", lazy="selectin")
```

Каждый объект стратегии загрузчика добавляет некоторую информацию к оператору, который будет использоваться позже сеансом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), когда он решает, как различные атрибуты должны быть загружены и/или как вести себя при доступе к ним.

В разделах ниже представлены некоторые из наиболее часто используемых стратегий загрузки.

{% hint style="info" %}
Смотри также:

Два раздела в методах загрузки отношений:

* [Configuring Loader Strategies at Mapping Time](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#relationship-lazy-option) — подробная информация о настройке стратегии для отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for)
* [Relationship Loading with Loader Options](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#relationship-loader-options) — подробности об использовании стратегий загрузчика во время запроса
{% endhint %}

### Загрузка selectin

Наиболее полезным загрузчиком в современной SQLAlchemy является опция загрузчика [selectinload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.selectinload). Эта опция решает наиболее распространенную форму проблемы «**N плюс один**», которая заключается в наборе объектов, которые ссылаются на связанные коллекции. [selectinload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.selectinload) гарантирует, что определенная коллекция для полной серии объектов загружается заранее с помощью одного запроса. Это делается с помощью формы **SELECT**, которая в большинстве случаев может быть отправлена только для связанной таблицы, без введения **JOIN** или подзапросов, и только для тех родительских объектов, для которых коллекция еще не загружена. Ниже мы иллюстрируем метод [selectinload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.selectinload), загружая все объекты **User** и все связанные с ними объекты **Address**; в то время как мы вызываем [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute) только один раз, учитывая конструкцию [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), при доступе к базе данных фактически выдаются два оператора **SELECT**, второй из которых предназначен для извлечения связанных объектов **Address**:

```python
>>> from sqlalchemy.orm import selectinload
>>> stmt = (
...   select(User).options(selectinload(User.addresses)).order_by(User.id)
... )
>>> for row in session.execute(stmt):
...     print(f"{row.User.name}  ({', '.join(a.email_address for a in row.User.addresses)})")
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account ORDER BY user_account.id
[...] ()
SELECT address.user_id AS address_user_id, address.id AS address_id,
address.email_address AS address_email_address
FROM address
WHERE address.user_id IN (?, ?, ?, ?, ?, ?)
[...] (1, 2, 3, 4, 5, 6)
```

```bash
spongebob  (spongebob@sqlalchemy.org)
sandy  (sandy@sqlalchemy.org, sandy@squirrelpower.org)
patrick  ()
squidward  ()
ehkrabs  ()
pkrabs  (pearl.krabs@gmail.com, pearl@aol.com)
```

{% hint style="info" %}
Смотри также:

[Select IN loading](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#selectin-eager-loading) - в документации [Relationship Loading Techniques](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html)
{% endhint %}

### Загрузка joined

Стратегия нетерпеливой загрузки [joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload) является старейшим нетерпеливым загрузчиком в SQLAlchemy, который дополняет оператор **SELECT**, передаваемый в базу данных, с помощью **JOIN** (которое может быть внешним или внутренним соединением в зависимости от параметров), которое затем может загружать связанные объекты.

Стратегия [joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload) лучше всего подходит для загрузки связанных объектов «**многие к одному**», так как для этого требуется только добавить дополнительные столбцы в строку основного объекта, которая будет выбрана в любом случае. Для большей эффективности он также принимает параметр [joinedload.innerjoin](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload.params.innerjoin), так что внутреннее соединение вместо внешнего может использоваться для случая, как показано ниже, когда мы знаем, что все объекты **Address** имеют связанного пользователя:

```python
>>> from sqlalchemy.orm import joinedload
>>> stmt = (
...   select(Address).options(joinedload(Address.user, innerjoin=True)).order_by(Address.id)
... )
>>> for row in session.execute(stmt):
...     print(f"{row.Address.email_address} {row.Address.user.name}")
```

```sql
SELECT address.id, address.email_address, address.user_id, user_account_1.id AS id_1,
user_account_1.name, user_account_1.fullname
FROM address
JOIN user_account AS user_account_1 ON user_account_1.id = address.user_id
ORDER BY address.id
[...] ()
```

```bash
spongebob@sqlalchemy.org spongebob
sandy@sqlalchemy.org sandy
sandy@squirrelpower.org sandy
pearl.krabs@gmail.com pkrabs
pearl@aol.com pkrabs
```

[joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload) также работает для коллекций, то есть отношений «один ко многим», однако _**имеет эффект умножения первичных строк на связанный элемент рекурсивным способом, что увеличивает количество данных, отправляемых для результирующего набора**_, на порядки величины для вложенных коллекций и/или более крупных коллекций, поэтому его использование по сравнению с другим параметром, таким как [selectinload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.selectinload), следует оценивать в каждом конкретном случае.

Важно отметить, что критерии **WHERE** и **ORDER BY** прилагаемого оператора [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select) не нацелены на таблицу, отображаемую методом **joinedload()**. Выше в SQL видно, что к таблице **user\_account** применяется анонимный псевдоним, который не может быть напрямую адресован в запросе. Эта концепция обсуждается более подробно в разделе «Дзен объединенной жадной загрузки» ([The Zen of Joined Eager Loading](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#zen-of-eager-loading)).

Предложение **ON**, отображаемое функцией [joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload), может быть затронуто непосредственно с помощью метода [PropComparator.and\_()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator.and\_), описанного ранее в разделе «[Дополнение критериев ON](rabota-so-svyazannymi-obektami.md#rasshirenie-kriteriev-on)»; примеры этого метода со стратегиями загрузчика приведены ниже в разделе «[Дополнительные пути стратегии загрузчика](rabota-so-svyazannymi-obektami.md#rasshirenie-putei-strategii-zagruzchika)». Однако в более общем смысле «**joined eager load**» может применяться к элементу [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), который использует [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join), используя подход, описанный в следующем разделе «[Явное присоединение + нетерпеливая загрузка](rabota-so-svyazannymi-obektami.md#yavnoe-obedinenie-join-+-neterpelivaya-zagruzka-eager-load)».

{% hint style="info" %}
Совет:

Важно отметить, что нетерпеливые загрузки «**многие к одному**» часто не нужны, поскольку проблема «**N плюс один**» в общем случае гораздо менее распространена. Когда все объекты ссылаются на один и тот же связанный объект, например, многие объекты **Address**, каждый из которых ссылается на одного и того же пользователя **User**, SQL будет выдан только один раз для этого объекта пользователя **User** с использованием обычной ленивой загрузки. Процедура ленивой загрузки будет искать связанный объект по первичному ключу в текущем сеансе [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), не выдавая никакого SQL, когда это возможно.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Joined Eager Loading](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#joined-eager-loading) - в документации [Relationship Loading Techniques](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html)
{% endhint %}

### Явное объединение JOIN + нетерпеливая загрузка EAGER LOAD

Если бы мы загружали строки **Address** при присоединении к таблице **user\_account**, используя такой метод, как [Select.join()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.join) для отображения **JOIN**, мы могли бы также использовать это **JOIN** для быстрой загрузки содержимого атрибута **Address.user** для каждого адреса возвращенного объекта. По сути, это то, что мы используем «joined eager loading», но сами рендерим **JOIN**. Этот распространенный вариант использования достигается с помощью параметра [contains\_eager()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.contains\_eager). Эта опция очень похожа на [joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload), за исключением того, что она предполагает, что мы сами настроили **JOIN**, и вместо этого указывает только на то, что дополнительные столбцы в предложении **COLUMNS** должны быть загружены в связанные атрибуты каждого возвращаемого объекта, например:

```python
>>> from sqlalchemy.orm import contains_eager
>>> stmt = (
...   select(Address).
...   join(Address.user).
...   where(User.name == 'pkrabs').
...   options(contains_eager(Address.user)).order_by(Address.id)
... )
>>> for row in session.execute(stmt):
...     print(f"{row.Address.email_address} {row.Address.user.name}")
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname,
address.id AS id_1, address.email_address, address.user_id
FROM address JOIN user_account ON user_account.id = address.user_id
WHERE user_account.name = ? ORDER BY address.id
[...] ('pkrabs',)
```

```bash
pearl.krabs@gmail.com pkrabs
pearl@aol.com pkrabs
```

Выше мы отфильтровали строки в **user\_account.name**, а также загрузили строки из **user\_account** в атрибут **Address.user** возвращенных строк. Если бы мы применяли [joinedload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload) отдельно, то получили бы SQL-запрос, который дважды соединялся бы без необходимости:

```python
>>> stmt = (
...   select(Address).
...   join(Address.user).
...   where(User.name == 'pkrabs').
...   options(joinedload(Address.user)).order_by(Address.id)
... )
>>> print(stmt)  # SELECT имеет JOIN и LEFT OUTER JOIN без необходимости
```

```sql
SELECT address.id, address.email_address, address.user_id,
user_account_1.id AS id_1, user_account_1.name, user_account_1.fullname
FROM address JOIN user_account ON user_account.id = address.user_id
LEFT OUTER JOIN user_account AS user_account_1 ON user_account_1.id = address.user_id
WHERE user_account.name = :name_1 ORDER BY address.id
```

{% hint style="info" %}
Смотри также:

Два раздела в методах загрузки отношений:

* [The Zen of Joined Eager Loading](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#zen-of-eager-loading) - подробно описывает вышеуказанную проблему.
* [Routing Explicit Joins/Statements into Eagerly Loaded Collections](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#contains-eager) — с помощью [contains\_eager()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.contains\_eager)
{% endhint %}

### Расширение путей стратегии загрузчика

В разделе «[Дополнение критериев ON](rabota-so-svyazannymi-obektami.md#rasshirenie-kriteriev-on)» мы показали, как добавлять произвольные критерии к **JOIN**, отображаемому с помощью отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), чтобы также включать дополнительные критерии в предложение **ON**. Метод [PropComparator.and\_()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator.and\_) на самом деле общедоступен для большинства опций загрузчика. Например, если мы хотим повторно загрузить имена пользователей и их адреса электронной почты, но опустить адреса электронной почты с доменом **sqlalchemy.org**, мы можем применить [PropComparator.and\_()](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator.and\_) к аргументу, переданному в [selectinload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.selectinload), чтобы ограничить этот критерий:

```python
>>> from sqlalchemy.orm import selectinload
>>> stmt = (
...   select(User).
...   options(
...       selectinload(
...           User.addresses.and_(
...             ~Address.email_address.endswith("sqlalchemy.org")
...           )
...       )
...   ).
...   order_by(User.id).
...   execution_options(populate_existing=True)
... )
>>> for row in session.execute(stmt):
...     print(f"{row.User.name}  ({', '.join(a.email_address for a in row.User.addresses)})")
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account ORDER BY user_account.id
[...] ()
SELECT address.user_id AS address_user_id, address.id AS address_id,
address.email_address AS address_email_address
FROM address
WHERE address.user_id IN (?, ?, ?, ?, ?, ?)
AND (address.email_address NOT LIKE '%' || ?)
[...] (1, 2, 3, 4, 5, 6, 'sqlalchemy.org')
```

```bash
spongebob  ()
sandy  (sandy@squirrelpower.org)
patrick  ()
squidward  ()
ehkrabs  ()
pkrabs  (pearl.krabs@gmail.com, pearl@aol.com)
```

Очень важно отметить выше, что специальная опция добавляется с `.execution_options(populate_existing=True)`. Эта опция, которая вступает в силу при выборке строк, указывает, что используемая нами опция загрузчика должна **заменить** существующее содержимое коллекций в объектах, если они уже загружены. Поскольку мы постоянно работаем с одним сеансом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), объекты, которые мы видим загружаемыми выше, являются теми же экземплярами Python, что и те, которые были впервые сохранены в начале раздела ORM этого руководства.

{% hint style="info" %}
Смотри также:

[Adding Criteria to loader options](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#loader-option-criteria) - в документации [Relationship Loading Techniques](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html)

[Populate Existing](https://docs.sqlalchemy.org/en/14/orm/queryguide.html#orm-queryguide-populate-existing) - in [ORM Querying Guide](https://docs.sqlalchemy.org/en/14/orm/queryguide.html)
{% endhint %}

### Raiseload

Стоит упомянуть еще одну стратегию загрузчика — [raiseload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.raiseload). Эта опция используется, чтобы полностью заблокировать приложение от проблем с N плюс один ([N plus one](https://docs.sqlalchemy.org/en/14/glossary.html#term-N-plus-one)), заставляя вместо этого то, что обычно было бы ленивой загрузкой, вызывать ошибку. Он имеет два варианта, которые контролируются с помощью параметра [raiseload.sql\_only](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.raiseload.params.sql\_only), чтобы блокировать либо ленивую загрузку, требующую SQL, либо все операции «**load**», включая те, которым нужно только обратиться к текущему сеансу [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session).

Один из способов использования [raiseload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.raiseload) — настроить его для самого отношения [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), установив для [relationship.lazy](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.lazy) значение «**raise\_on\_sql**», чтобы для конкретного сопоставления определенное отношение никогда не пыталось выдать SQL:

```python
class User(Base):
    __tablename__ = 'user_account'

    # ... Column mappings

    addresses = relationship("Address", back_populates="user", lazy="raise_on_sql")


class Address(Base):
    __tablename__ = 'address'

    # ... Column mappings

    user = relationship("User", back_populates="addresses", lazy="raise_on_sql")
```

Используя такое сопоставление, **приложение блокируется от ленивой загрузки**, что указывает на то, что для конкретного запроса необходимо указать стратегию загрузчика:

```python
u1 = s.execute(select(User)).scalars().first()
u1.addresses
sqlalchemy.exc.InvalidRequestError: 'User.addresses' is not available
due to lazy='raise_on_sql'
```

Исключение будет означать, что эта коллекция **должна быть загружена заранее**:

```python
u1 = s.execute(select(User).options(selectinload(User.addresses))).scalars().first()
```

Параметр `lazy="raise_on_sql"` также пытается быть умным в отношениях "**многие к одному**"; выше, если атрибут **Address.user** объекта **Address** не был загружен, но этот объект **User** локально присутствовал в том же сеансе [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), стратегия «**raiseload**» не вызвала бы ошибку.

{% hint style="info" %}
Смотри также:

[Preventing unwanted lazy loads using raiseload](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#prevent-lazy-with-raiseload) - в документации [Relationship Loading Techniques](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html)
{% endhint %}

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел руководства: [Дополнительная литература](dopolnitelnoe-chtenie.md)
{% endhint %}
