# Настройка доступа к коллекции

Сопоставление отношений «один ко многим» или «многие ко многим» приводит к набору значений, доступных через атрибут в родительском экземпляре. По умолчанию эта коллекция представляет собой список **list**:

```python
class Parent(Base):
    __tablename__ = 'parent'
    parent_id = Column(Integer, primary_key=True)

    children = relationship(Child)

parent = Parent()
parent.children.append(Child())
print(parent.children[0])
```

Коллекции не ограничиваются списками. Множества, изменяемые последовательности и почти любой другой объект Python, который может выступать в качестве контейнера, можно использовать вместо списка по умолчанию, указав параметр [relationship.collection\_class](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.collection\_class) в отношении [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship):

```python
class Parent(Base):
    __tablename__ = 'parent'
    parent_id = Column(Integer, primary_key=True)

    # использование множества
    children = relationship(Child, collection_class=set)

parent = Parent()
child = Child()
parent.children.add(child)
assert child in parent.children
```

## Коллекции словарей

При использовании словаря в качестве коллекции требуется небольшая дополнительная информация. Это связано с тем, что объекты всегда загружаются из базы данных в виде списков, и для правильного заполнения словаря должна быть доступна стратегия генерации ключей. Функция [attribute\_mapped\_collection()](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.attribute\_mapped\_collection) на сегодняшний день является наиболее распространенным способом создания простой коллекции словарей. Он**а** создает класс словаря, который будет применять определенный атрибут сопоставленного класса в качестве ключа. Ниже мы сопоставляем класс **Item**, содержащий словарь элементов **Note**, связанных с атрибутом **Note.keyword**:

```python
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.orm.collections import attribute_mapped_collection
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Item(Base):
    __tablename__ = 'item'
    id = Column(Integer, primary_key=True)
    notes = relationship("Note",
                collection_class=attribute_mapped_collection('keyword'),
                cascade="all, delete-orphan")

class Note(Base):
    __tablename__ = 'note'
    id = Column(Integer, primary_key=True)
    item_id = Column(Integer, ForeignKey('item.id'), nullable=False)
    keyword = Column(String)
    text = Column(String)

    def __init__(self, keyword, text):
        self.keyword = keyword
        self.text = text
```

**Item.notes** — это словарь:

```python
>>> item = Item()
>>> item.notes['a'] = Note('a', 'atext')
>>> item.notes.items()
{'a': <__main__.Note object at 0x2eaaf0>}
```

[attribute\_mapped\_collection()](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.attribute\_mapped\_collection) гарантирует, что атрибут `.keyword` каждой заметки **Note** соответствует ключу в словаре. Например, при назначении **Item.notes** ключ словаря, который мы предоставляем, должен соответствовать ключу фактического объекта **Note**:

```python
item = Item()
item.notes = {
            'a': Note('a', 'atext'),
            'b': Note('b', 'btext')
        }
```

Атрибут, который [attribute\_mapped\_collection()](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.attribute\_mapped\_collection) использует в качестве ключа, вообще не нужно сопоставлять! Использование обычного Python `@property` позволяет использовать в качестве ключа практически любую информацию или комбинацию сведений об объекте, как показано ниже, когда мы устанавливаем его как кортеж **Note.keyword** и первые десять букв поля **Note.text**:

```python
class Item(Base):
    __tablename__ = 'item'
    id = Column(Integer, primary_key=True)
    notes = relationship("Note",
                collection_class=attribute_mapped_collection('note_key'),
                backref="item",
                cascade="all, delete-orphan")

class Note(Base):
    __tablename__ = 'note'
    id = Column(Integer, primary_key=True)
    item_id = Column(Integer, ForeignKey('item.id'), nullable=False)
    keyword = Column(String)
    text = Column(String)

    @property
    def note_key(self):
        return (self.keyword, self.text[0:10])

    def __init__(self, keyword, text):
        self.keyword = keyword
        self.text = text
```

Выше мы добавили обратную ссылку **Note.item**. Присваивая это обратное отношение, **Note** добавляется в словарь **Item.notes** и ключ генерируется для нас автоматически:

```python
>>> item = Item()
>>> n1 = Note("a", "atext")
>>> n1.item = item
>>> item.notes
{('a', 'atext'): <__main__.Note object at 0x2eaaf0>}
```

Другие встроенные типы словарей включают [column\_mapped\_collection()](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.column\_mapped\_collection), который почти похож на [attribute\_mapped\_collection()](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.attribute\_mapped\_collection), за исключением того, что он напрямую связан с объектом [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column):

```python
from sqlalchemy.orm.collections import column_mapped_collection

class Item(Base):
    __tablename__ = 'item'
    id = Column(Integer, primary_key=True)
    notes = relationship("Note",
                collection_class=column_mapped_collection(Note.__table__.c.keyword),
                cascade="all, delete-orphan")
```

а также [mapped\_collection()](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.mapped\_collection), которому передается любая вызываемая функция. Обратите внимание, что обычно проще использовать [attribute\_mapped\_collection()](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.attribute\_mapped\_collection) вместе с `@property`, как упоминалось ранее:

```python
from sqlalchemy.orm.collections import mapped_collection

class Item(Base):
    __tablename__ = 'item'
    id = Column(Integer, primary_key=True)
    notes = relationship("Note",
                collection_class=mapped_collection(lambda note: note.text[0:10]),
                cascade="all, delete-orphan")
```

Сопоставления словарей часто сочетаются с расширением «**Association Proxy**» для создания оптимизированных представлений словарей. Примеры см. в разделе «Проксирование коллекций на основе словарей» ([Proxying to Dictionary Based Collections](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html#proxying-dictionaries)) и «Композитные прокси-ассоциации» ([Composite Association Proxies](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html#composite-association-proxy)).

### Работа с ключевыми мутациями и обратное заполнение коллекций словарей

При использовании [attribute\_mapped\_collection()](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.attribute\_mapped\_collection) ключ «**key**» для словаря берется из атрибута целевого объекта. **Изменения этого ключа не отслеживаются**. Это означает, что ключ должен быть назначен при первом использовании, и если ключ изменится, коллекция не будет видоизменена. Типичный пример, когда это может быть проблемой, — использование обратных ссылок для заполнения коллекции сопоставленных атрибутов. Учитывая следующее:

```python
class A(Base):
    __tablename__ = "a"

    id = Column(Integer, primary_key=True)
    bs = relationship(
        "B",
        collection_class=attribute_mapped_collection("data"),
        back_populates="a",
    )


class B(Base):
    __tablename__ = "b"
    id = Column(Integer, primary_key=True)
    a_id = Column(ForeignKey("a.id"))
    data = Column(String)

    a = relationship("A", back_populates="bs")
```

Выше, если мы создадим **B()**, который ссылается на конкретный **A()**, обратное заполнение добавит **B()** в коллекцию **A.bs**, однако, если значение **B.data** еще не установлено, ключ будет `None`:

```python
>>> a1 = A()
>>> b1 = B(a=a1)
>>> a1.bs
{None: <test3.B object at 0x7f7b1023ef70>}
```

Установка **b1.data** постфактум не обновляет коллекцию:

```python
>>> b1.data = 'the key'
>>> a1.bs
{None: <test3.B object at 0x7f7b1023ef70>}
```

Это также можно увидеть, если попытаться настроить **B()** в конструкторе. Порядок аргументов изменяет результат:

```python
>>> B(a=a1, data='the key')
<test3.B object at 0x7f7b10114280>
>>> a1.bs
{None: <test3.B object at 0x7f7b10114280>}
```

против:

```python
>>> B(data='the key', a=a1)
<test3.B object at 0x7f7b10114340>
>>> a1.bs
{'the key': <test3.B object at 0x7f7b10114340>}
```

Если обратные ссылки используются таким образом, убедитесь, что атрибуты заполняются в правильном порядке, используя метод **\_\_init\_\_**.

Обработчик событий, такой как следующий, также может использоваться для отслеживания изменений в коллекции:

```python
from sqlalchemy import event

from sqlalchemy.orm import attributes

@event.listens_for(B.data, "set")
def set_item(obj, value, previous, initiator):
    if obj.a is not None:
        previous = None if previous == attributes.NO_VALUE else previous
        obj.a.bs[value] = obj
        obj.a.bs.pop(previous)
```

| Название объекта                                                                                                                                             | Описание                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| [attribute\_mapped\_collection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.attribute\_mapped\_collection)(attr\_name) | Тип коллекции на основе словаря с ключами на основе атрибутов. |
| [column\_mapped\_collection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.column\_mapped\_collection)(mapping\_spec)    | Тип коллекции на основе словаря с ключами на основе столбцов.  |
| [mapped\_collection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.mapped\_collection)(keyfunc)                          | Тип коллекции на основе словаря с произвольным набором ключей. |

### _function_ sqlalchemy.orm.collections.attribute\_mapped\_collection(_attr\_name_)

Тип коллекции на основе словаря с ключами на основе атрибутов.

Возвращает фабрику [MappedCollection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.MappedCollection) с ключом на основе атрибута **«attr\_name»** сущностей в коллекции, где **attr\_name** — строковое имя атрибута.

{% hint style="danger" %}
значение ключа должно быть присвоено его окончательному значению **до того**, как к нему будет осуществлен доступ из сопоставленной коллекции атрибутов. Кроме того, изменения атрибута ключа **не отслеживаются** автоматически, что означает, что ключ в словаре не синхронизируется автоматически со значением ключа в самом целевом объекте. Пример см. в разделе [Работа с ключевыми мутациями и обратное заполнение коллекций словарей](nastroika-dostupa-k-kollekcii.md#rabota-s-klyuchevymi-mutaciyami-i-obratnoe-zapolnenie-kollekcii-slovarei).
{% endhint %}

### _function_ sqlalchemy.orm.collections.column\_mapped\_collection(_mapping\_spec_)

Тип коллекции на основе словаря с ключами на основе столбцов.

Возвращает фабрику [MappedCollection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.MappedCollection) с ключевой функцией, сгенерированной из **mapping\_spec**, которая может быть столбцом или последовательностью столбцов.

Значение ключа должно быть неизменным в течение всего времени существования объекта. Вы не можете, например, сопоставить значения внешнего ключа, если эти значения ключа изменятся во время сеанса, то есть с `None` на целое число, назначенное базой данных после очистки сеанса.

### _function_ sqlalchemy.orm.collections.mapped\_collection(_keyfunc_)

Тип коллекции на основе словаря с произвольным набором ключей.

Возвращает фабрику [MappedCollection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.MappedCollection) с ключевой функцией, созданной из **keyfunc**, вызываемого объекта, который принимает сущность и возвращает значение ключа.

Значение ключа должно быть неизменным в течение всего времени существования объекта. Вы не можете, например, сопоставить значения внешнего ключа, если эти значения ключа изменятся во время сеанса, то есть с `None` на целое число, назначенное базой данных после очистки сеанса.
