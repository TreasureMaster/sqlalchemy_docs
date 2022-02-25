# Связывание отношений с backref

## Связывание отношений с backref

Аргумент ключевого слова relationship.backref был впервые представлен в [учебнике по реляционным объектам (API 1.x)](../uchebnoe-posobie-po-relyacionnym-obektam-1.x-api/) и упоминался во многих приведенных здесь примерах. Что он на самом деле делает? Начнем с канонического сценария пользователя **User** и адреса **Address**:

```python
from sqlalchemy import Integer, ForeignKey, String, Column
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    name = Column(String)

    addresses = relationship("Address", backref="user")

class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    email = Column(String)
    user_id = Column(Integer, ForeignKey('user.id'))
```

Приведенная выше конфигурация устанавливает коллекцию объектов **Address** для пользователя **User,** именованную как **User.addresses**. Он также устанавливает атрибут `.user` для **Address**, который будет ссылаться на родительский объект **User**.

На самом деле ключевое слово [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) — это всего лишь _**общий ярлык для размещения второго отношения**_ [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) в отображении **Address**, включая создание прослушивателя событий с обеих сторон, который будет отражать операции с атрибутами в обоих направлениях. Приведенная выше конфигурация эквивалентна:

```python
from sqlalchemy import Integer, ForeignKey, String, Column
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    name = Column(String)

    addresses = relationship("Address", back_populates="user")

class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    email = Column(String)
    user_id = Column(Integer, ForeignKey('user.id'))

    user = relationship("User", back_populates="addresses")
```

Выше мы явно добавляем отношение `.user` к **Address**. В обоих отношениях директива [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) сообщает каждому отношению о другом, указывая, что они должны установить «двунаправленное» поведение друг с другом. Основным эффектом этой конфигурации является то, что отношение добавляет обработчики событий к обоим атрибутам, которые имеют поведение _**«когда здесь происходит событие добавления или установки, мы устанавливаем себя на входящий атрибут, используя это конкретное имя атрибута»**_. Поведение иллюстрируется следующим образом. Начните с экземпляра пользователя **User** и адреса **Address**. Коллекция `.addresses` пуста, а атрибут `.user` равен `None`:

```python
>>> u1 = User()
>>> a1 = Address()
>>> u1.addresses
[]
>>> print(a1.user)
None
```

Однако после добавления **Address** к коллекции **u1.addresses** заполняется и коллекция, и скалярный атрибут:

```python
>>> u1.addresses.append(a1)
>>> u1.addresses
[<__main__.Address object at 0x12a6ed0>]
>>> a1.user
<__main__.User object at 0x12a6590>
```

Это поведение, конечно, работает и в обратном порядке для операций удаления, а также для эквивалентных операций с обеих сторон. Например, когда для `.user` снова установлено значение `None`, объект **Address** удаляется из обратной коллекции:

```python
>>> a1.user = None
>>> u1.addresses
[]
```

Манипуляции с коллекцией `.addresses` и атрибутом `.user` полностью выполняются в Python без какого-либо взаимодействия с базой данных SQL. Без этого поведения правильное состояние было бы очевидным с обеих сторон после того, как данные были сброшены в базу данных, а затем перезагружены после выполнения операции фиксации или истечения срока действия. Преимущество поведения [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) / [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) состоит в том, что _**обычные двунаправленные операции могут отражать правильное состояние, не требуя обращения к базе данных**_.

Помните, что когда ключевое слово [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) используется для одного отношения, это точно так же, как если бы два вышеупомянутых отношения были созданы по отдельности с использованием [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) для каждого.

### Аргументы backref

Мы установили, что ключевое слово [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) — это просто ярлык для создания двух отдельных конструкций [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), которые ссылаются друг на друга. Частью поведения этого ярлыка является то, что определенные конфигурационные аргументы, применяемые к отношениям [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), также будут применяться в другом направлении, а именно к тем аргументам, которые описывают отношения на уровне схемы и вряд ли будут отличаться в обратном направлении. Обычным случаем здесь является отношение «многие ко многим» [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), которое имеет аргумент «[relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary)», или «один ко многим» или «многие к одному», которое имеет аргумент «[relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin)» (аргумент [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) обсуждается в разделе «Указание альтернативных условий соединения» ([Specifying Alternate Join Conditions](https://docs.sqlalchemy.org/en/14/orm/join\_conditions.html#relationship-primaryjoin))). Например, если бы мы ограничили список объектов **Address** теми, которые начинаются с «tony»:

```python
from sqlalchemy import Integer, ForeignKey, String, Column
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    name = Column(String)

    addresses = relationship("Address",
                    primaryjoin="and_(User.id==Address.user_id, "
                        "Address.email.startswith('tony'))",
                    backref="user")

class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    email = Column(String)
    user_id = Column(Integer, ForeignKey('user.id'))
```

Изучив результирующее свойство, мы можем заметить, что к обеим сторонам отношения применяется это условие соединения:

```python
>>> print(User.addresses.property.primaryjoin)
"user".id = address.user_id AND address.email LIKE :email_1 || '%%'
>>>
>>> print(Address.user.property.primaryjoin)
"user".id = address.user_id AND address.email LIKE :email_1 || '%%'
>>>
```

Это повторное использование аргументов должно в значительной степени делать «правильные вещи» — оно использует только применимые аргументы, а в случае отношения «многие ко многим» будет использовать [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) и [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondaryjoin) в другом направлении (для этого см. пример в Самореферентных отношениях «многие ко многим» ([Self-Referential Many-to-Many Relationship](https://docs.sqlalchemy.org/en/14/orm/join\_conditions.html#self-referential-many-to-many))).

Однако очень часто бывает так, что мы хотели бы указать аргументы, относящиеся только к той стороне, где мы разместили «обратную ссылку». Это включает аргументы отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), такие как [relationship.lazy](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.lazy), [relationship.remote\_side](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.remote\_side), [relationship.cascade](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.cascade) и [relationship.cascade\_backrefs](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.cascade\_backrefs). В этом случае мы используем функцию [backref()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.backref) вместо строки:

```python
# <other imports>
from sqlalchemy.orm import backref

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    name = Column(String)

    addresses = relationship("Address",
                    backref=backref("user", lazy="joined"))
```

Там, где выше, мы поместили директиву `lazy="joined"` только на стороне **Address.user**, указав, что при выполнении запроса к **Address**, соединение с сущностью **User** должно выполняться автоматически, что заполнит атрибут `.user` каждого возвращенного **Address**. Функция [backref()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.backref) отформатировала аргументы, которые мы ей передали, в форму, которая интерпретируется принимающим отношением [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) как дополнительные аргументы, применяемые к новому отношению, которое оно создает.

### Настройка каскада для обратных ссылок backrefs

Ключевое поведение, которое происходит в серии 1.x SQLAlchemy в отношении обратных ссылок, заключается в том, что по умолчанию каскады ([cascades](https://docs.sqlalchemy.org/en/14/orm/cascades.html#unitofwork-cascades)) будут выполняться в двух направлениях. В основном это означает, что если вы начинаете с объекта **User**, который был сохранен в сеансе [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session):

```python
user = session.query(User).filter(User.id == 1).first()
```

Вышеупомянутый пользователь **User** является постоянным ([persistent](https://docs.sqlalchemy.org/en/14/glossary.html#term-persistent)) в сеансе [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session). Обычно интуитивно понятно, что если мы создадим объект **Address** и добавим его в коллекцию **User.addresses**, он автоматически добавится в [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), как в примере ниже:

```python
user = session.query(User).filter(User.id == 1).first()
address = Address(email_address='foo')
user.addresses.append(address)
```

Описанное выше поведение известно как «каскад сохранения обновлений» и описано в разделе «Каскады» ([Cascades](https://docs.sqlalchemy.org/en/14/orm/cascades.html#unitofwork-cascades)).

Однако если вместо этого мы создадим новый объект **Address** и свяжем объект **User** с адресом следующим образом:

```python
address = Address(email_address='foo', user=user)
```

В приведенном выше примере _**не так интуитивно понятно**_, что адрес автоматически добавляется к сеансу [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session). Однако поведение обратной ссылки **Address.user** указывает, что объект **Address** также добавлен в коллекцию **User.addresses**. Это, в свою очередь, инициирует _**каскадную операцию**_, которая указывает, что этот адрес **Address** должен быть помещен в сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) как ожидающий объект ([pending](https://docs.sqlalchemy.org/en/14/glossary.html#term-pending)).

Так как это поведение было определено как противоречащее здравому смыслу для большинства людей, его можно отключить, установив для свойства [relationship.cascade\_backrefs](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.cascade\_backrefs) значение `False`, например:

```python
class User(Base):
    # ...

    addresses = relationship(
       "Address",
       back_populates="user",
       cascade_backrefs=False
    )
```

См. пример в разделе «Управление каскадом на обратных ссылках» ([Controlling Cascade on Backrefs](https://docs.sqlalchemy.org/en/14/orm/cascades.html#backref-cascade)) для получения дополнительной информации.

{% hint style="info" %}
Смотри также:

[Controlling Cascade on Backrefs](https://docs.sqlalchemy.org/en/14/orm/cascades.html#backref-cascade)
{% endhint %}

### Односторонние обратные ссылки (backrefs)

Необычным случаем является «односторонняя обратная ссылка». Именно здесь поведение «обратного заполнения» обратной ссылки желательно только в одном направлении. Примером этого является коллекция, которая содержит фильтрующее условие [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin). Мы хотели бы добавлять элементы в эту коллекцию по мере необходимости и заполнять ими «родительский» объект входящего объекта. Однако мы также хотели бы иметь элементы, которые не являются частью коллекции, но все же имеют одну и ту же «родительскую» ассоциацию — эти элементы никогда не должны быть в коллекции.

Возьмем наш предыдущий пример, где мы установили [relationship.primaryjoin](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin), который ограничивал коллекцию только объектами **Address**, чей адрес электронной почты начинался со слова _**tony**_, обычное поведение обратной ссылки заключается в том, что все элементы заполняются в обоих направлениях. Мы бы не хотели такого поведения в случае, подобном следующему:

```python
>>> u1 = User()
>>> a1 = Address(email='mary')
>>> a1.user = u1
>>> u1.addresses
[<__main__.Address object at 0x1411910>]
```

Выше объект **Address**, который не соответствует критерию «начинается с «tony»», присутствует в коллекции адресов **u1**. После сброса этих объектов, фиксации транзакции и истечения срока действия их атрибутов для повторной загрузки коллекция **addresses** попадет в базу данных при следующем доступе, и этот объект **Address** больше не будет присутствовать из-за условия фильтрации. Но мы можем покончить с этой нежелательной стороной поведения «обратной ссылки» на стороне Python, используя две отдельные конструкции отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), разместив [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) только с одной стороны:

```python
from sqlalchemy import Integer, ForeignKey, String, Column
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    name = Column(String)
    addresses = relationship("Address",
                    primaryjoin="and_(User.id==Address.user_id, "
                        "Address.email.startswith('tony'))",
                    back_populates="user")

class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    email = Column(String)
    user_id = Column(Integer, ForeignKey('user.id'))
    user = relationship("User")
```

В приведенном выше сценарии добавление объекта **Address** к коллекции `.addresses` пользователя **User** всегда будет устанавливать атрибут `.user` для этого адреса **Address**:

```python
>>> u1 = User()
>>> a1 = Address(email='tony')
>>> u1.addresses.append(a1)
>>> a1.user
<__main__.User object at 0x1411850>
```

Однако применение пользователя **User** к атрибуту `.user` адреса **Address** не добавит объект **Address** в коллекцию:

```python
>>> a2 = Address(email='mary')
>>> a2.user = u1
>>> a2 in u1.addresses
False
```

Конечно, здесь мы отключили часть полезности отношения [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref), поскольку, когда мы добавляем адрес **Address**, который соответствует критериям `email.startswith('tony')`, он не будет отображаться в коллекции **User.addresses** до тех пор, пока сеанс не будет сброшен в БД, а атрибуты не будут перезагружены после операции фиксации или истечения срока действия. Хотя мы могли бы рассмотреть событие атрибута, которое проверяет этот критерий в Python, это начинает пересекать черту слишком большого дублирования поведения SQL в Python. Само поведение обратной ссылки является лишь небольшим нарушением этой философии - SQLAlchemy пытается свести их к минимуму в целом.
