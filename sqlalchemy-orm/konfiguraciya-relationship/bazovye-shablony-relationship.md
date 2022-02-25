# Базовые шаблоны Relationship

Краткое пошаговое руководство по основным реляционным шаблонам.

Импорт, используемый для каждого из следующих разделов, выглядит следующим образом:

```python
from sqlalchemy import Table, Column, Integer, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```

## Один ко многим (One To Many)

Связь «один ко многим» помещает внешний ключ в дочернюю таблицу, ссылающуюся на родительскую. [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) затем указывается в родительском объекте как ссылка на набор элементов, представленных дочерним элементом:

```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    children = relationship("Child")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
```

Чтобы установить двунаправленное отношение «один-ко-многим», где «обратная» сторона — это «многие к одному», укажите дополнительное [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) и соедините их с помощью параметр [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates):

```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    children = relationship("Child", back_populates="parent")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
    parent = relationship("Parent", back_populates="children")
```

**Child** получит родительский атрибут **parent** с семантикой «многие к одному».

В качестве альтернативы, параметр [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) может использоваться для одного [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) вместо использования [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates):

```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    children = relationship("Child", backref="parent")
```

### Настройка поведения удаления для одного ко многим

Часто бывает так, что все дочерние объекты **Child** должны быть удалены, когда удаляется их родительский объект **Parent**. Для настройки этого поведения используется параметр каскадного удаления **delete**, описанный в разделе [delete](https://docs.sqlalchemy.org/en/14/orm/cascades.html#cascade-delete). Дополнительным параметром является то, что дочерний объект **Child** сам может быть удален, когда он отсоединяется от своего родителя. Это поведение описано в [delete-orphan](https://docs.sqlalchemy.org/en/14/orm/cascades.html#cascade-delete-orphan).

{% hint style="info" %}
Смотри также:

[delete](https://docs.sqlalchemy.org/en/14/orm/cascades.html#cascade-delete)

[Using foreign key ON DELETE cascade with ORM relationships](https://docs.sqlalchemy.org/en/14/orm/cascades.html#passive-deletes)

[delete-orphan](https://docs.sqlalchemy.org/en/14/orm/cascades.html#cascade-delete-orphan)
{% endhint %}

## Многие ко одному (Many To One)

Многие к одному помещает внешний ключ в родительскую таблицу, ссылающуюся на дочернюю. [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) объявляется в родительском элементе, где будет создан новый скалярно-удерживающий атрибут:

```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    child_id = Column(Integer, ForeignKey('child.id'))
    child = relationship("Child")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
```

Двунаправленное поведение достигается путем добавления второго [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) и применения параметра [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) в обоих направлениях:

```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    child_id = Column(Integer, ForeignKey('child.id'))
    child = relationship("Child", back_populates="parents")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parents = relationship("Parent", back_populates="child")
```

В качестве альтернативы, параметр [relationshp.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) может быть применен к одному [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), такому как **Parent.child**:

```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    child_id = Column(Integer, ForeignKey('child.id'))
    child = relationship("Child", backref="parents")
```

### Один ко одному (One To One)

«Один к одному» — это, по сути, двунаправленная связь со скалярным атрибутом с обеих сторон. В ORM «один к одному» считается соглашением, при котором ORM ожидает, что для любой родительской строки будет существовать только одна связанная строка.

Соглашение «один-к-одному» достигается путем применения значения `False` к параметру [relationship.uselist](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.uselist) конструкции [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) или, в некоторых случаях, к конструкции [backref()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.backref), применяя ее к конструкции «один-ко-многим», или «коллекционная» сторона отношений.

В приведенном ниже примере мы представляем двунаправленную связь, которая включает отношения «[один ко многим](bazovye-shablony-relationship.md#odin-ko-mnogim-one-to-many)» (`Parent.children`) и «[многие к одному](bazovye-shablony-relationship.md#mnogie-ko-odnomu-many-to-one)» (`Child.parent`):

```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)

    # коллекция one-to-many
    children = relationship("Child", back_populates="parent")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))

    # скаляр many-to-one
    parent = relationship("Parent", back_populates="children")
```

Выше `Parent.children` — это сторона «один ко многим», относящаяся к коллекции, а `Child.parent` — это сторона «многие к одному», относящаяся к одному объекту. Чтобы преобразовать это в «один к одному», сторона «один ко многим» или «коллекция» преобразуется в скалярное отношение с использованием флага `uselist=False`, переименовывая `Parent.children` в `Parent.child` для ясности:

```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)

    # первоначальный one-to-many Parent.children - это сейчас
    # one-to-one Parent.child
    child = relationship("Child", back_populates="parent", uselist=False)

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))

    # сторона many-to-one остается, см. совет ниже
    parent = relationship("Parent", back_populates="child")
```

Выше, когда мы загружаем родительский объект **Parent**, атрибут **Parent.child** будет ссылаться на один дочерний объект **Child**, а не на коллекцию. Если мы заменим значение **Parent.child** новым дочерним объектом **Child**, единица рабочего процесса ORM заменит предыдущую дочернюю строку новой, установив для предыдущего столбца **child.parent\_id** значение **NULL** по умолчанию, если только не установлены определенные [каскадные](https://docs.sqlalchemy.org/en/14/orm/cascades.html#unitofwork-cascades) поведения.

{% hint style="success" %}
Совет:

Как упоминалось ранее, ORM рассматривает шаблон «один-к-одному» как соглашение, где он делает предположение, что при загрузке атрибута **Parent.child** в родительском объекте **Parent** будет возвращена только одна строка. Если будет возвращено более одной строки, ORM выдаст предупреждение.

Однако сторона **Child.parent** вышеупомянутого отношения остается как отношение «многие к одному» и не изменяется, и в самой ORM нет встроенной системы, которая предотвращает создание более одного дочернего объекта **Child** против одного и того же родителя **Parent** во время настойчивости. Вместо этого в фактической схеме базы данных могут использоваться такие методы, как ограничения уникальности ([unique constraints](https://docs.sqlalchemy.org/en/14/core/constraints.html#schema-unique-constraint)), чтобы обеспечить такое расположение, где ограничение уникальности в столбце **Child.parent\_id** гарантирует, что только одна дочерняя строка **Child** может ссылаться на конкретную родительскую строку **Parent** в каждый момент времени.
{% endhint %}

В случае, когда параметр [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) используется для определения стороны «один-ко-многим», его можно преобразовать в соглашение «один-к-одному» с помощью функции [backref()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.backref), которая разрешает связь, сгенерированную [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) для получения пользовательских параметров, в данном случае параметра списка использования:

```python
from sqlalchemy.orm import backref

class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
    parent = relationship("Parent", backref=backref("child", uselist=False))
```

## Многие ко многим (Many To Many)

Многие ко многим добавляет ассоциативную таблицу между двумя классами. Таблица ассоциаций указана аргументом [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary) для [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship). Обычно [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) использует объект [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), связанный с декларативным базовым классом, чтобы директивы [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey) могли найти удаленные таблицы, с которыми можно связать:

```python
association_table = Table('association', Base.metadata,
    Column('left_id', ForeignKey('left.id')),
    Column('right_id', ForeignKey('right.id'))
)

class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)
    children = relationship("Child",
                    secondary=association_table)

class Child(Base):
    __tablename__ = 'right'
    id = Column(Integer, primary_key=True)
```

{% hint style="success" %}
Совет:

В приведенной выше «таблице ассоциаций» установлены ограничения внешнего ключа, которые относятся к двум таблицам сущностей по обе стороны отношения. Тип данных каждого из **association.left\_id** и **association.right\_id** обычно выводится из типа данных таблицы, на которую ссылаются, и может быть опущен. Также **рекомендуется**, хотя это никоим образом не требуется SQLAlchemy, чтобы столбцы, которые ссылаются на две таблицы сущностей, устанавливались либо в рамках **уникального ограничения**, либо, что чаще встречается, в качестве **ограничения первичного ключа**; это гарантирует, что повторяющиеся строки не будут сохраняться в таблице независимо от проблем на стороне приложения:

```python
association_table = Table('association', Base.metadata,
    Column('left_id', ForeignKey('left.id'), primary_key=True),
    Column('right_id', ForeignKey('right.id'), primary_key=True)
)
```
{% endhint %}

Для двунаправленного отношения обе стороны отношения содержат коллекцию. Укажите с помощью [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates), и для каждого [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) общую таблицу ассоциаций:

```python
association_table = Table('association', Base.metadata,
    Column('left_id', ForeignKey('left.id'), primary_key=True),
    Column('right_id', ForeignKey('right.id'), primary_key=True)
)

class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)
    children = relationship(
        "Child",
        secondary=association_table,
        back_populates="parents")

class Child(Base):
    __tablename__ = 'right'
    id = Column(Integer, primary_key=True)
    parents = relationship(
        "Parent",
        secondary=association_table,
        back_populates="children")
```

При использовании параметра [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) вместо [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) обратная ссылка будет автоматически использовать тот же аргумент [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary) для обратного отношения:

```python
association_table = Table('association', Base.metadata,
    Column('left_id', ForeignKey('left.id'), primary_key=True),
    Column('right_id', ForeignKey('right.id'), primary_key=True)
)

class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)
    children = relationship("Child",
                    secondary=association_table,
                    backref="parents")

class Child(Base):
    __tablename__ = 'right'
    id = Column(Integer, primary_key=True)
```

Аргумент [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary) для [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) также принимает вызываемый объект, который возвращает окончательный аргумент, который оценивается только при первом использовании _mappers_. Используя это, мы можем определить **association\_table** на более позднем этапе, если она доступна для вызываемого объекта после завершения инициализации всех модулей:

```python
class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)
    children = relationship("Child",
                    secondary=lambda: association_table,
                    backref="parents")
```

При использовании декларативного расширения также принимается традиционное «строковое имя таблицы», соответствующее имени таблицы, хранящемуся в **Base.metadata.tables**:

```python
class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)
    children = relationship("Child",
                    secondary="association",
                    backref="parents")
```

{% hint style="danger" %}
При передаче в виде строки, которую может вычислить Python, аргумент [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary) интерпретируется с помощью функции Python **eval()**. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ**. Подробную информацию о декларативной оценке аргументов [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

### Удаление строк из таблицы «многие ко многим»

Поведение, которое является уникальным для аргумента [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary) для [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), заключается в том, что таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), указанная здесь, автоматически подвергается операторам **INSERT** и **DELETE**, когда объекты добавляются или удаляются из коллекции. **Нет необходимости удалять из этой таблицы вручную**. Действие удаления записи из коллекции приведет к удалению строки при сбросе:

```python
# строка будет удалена из таблицы "secondary"
# автоматически
myparent.children.remove(somechild)
```

Часто возникает вопрос, как можно удалить строку во «вторичной» таблице, когда дочерний объект передается непосредственно в [Session.delete()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.delete):

```python
session.delete(somechild)
```

Здесь есть несколько возможностей:

* Если существует [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) от **Parent** к **Child**, но **нет обратной связи**, которая связывает конкретный **Child** с каждым **Parent**, SQLAlchemy не будет знать, что при удалении этого конкретного **Child** ему необходимо поддерживать «вторичный объект», которая связывает его с родительским **Parent**. Никакого удаления «вторичной» таблицы не произойдет.
* Если существует отношение, которое связывает конкретный **Child** с каждым **Parent**, предположим, что оно называется **Child.parents**, SQLAlchemy по умолчанию загрузит коллекцию **Child.parents**, чтобы найти все родительские объекты **Parent**, и удалит каждую строку из «вторичной» таблицы, которая устанавливает эта ссылка. Обратите внимание, что эта связь не обязательно должна быть двунаправленной; SQLAlchemy строго следит за всеми отношениями [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), связанными с удаляемым дочерним объектом **Child**.
* Здесь более эффективным вариантом является использование директив **ON DELETE CASCADE** с внешними ключами, используемыми базой данных. Предполагая, что база данных поддерживает эту функцию, можно настроить саму базу данных на автоматическое удаление строк в «дополнительной» таблице по мере удаления ссылающихся строк в «дочерней». В этом случае SQLAlchemy можно дать указание отказаться от активной загрузки коллекции **Child.parents**, используя директиву [relationship.passive\_deletes](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.passive\_deletes) для отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship); см. Использование каскада внешнего ключа ON DELETE с отношениями ORM ([Using foreign key ON DELETE cascade with ORM relationships](https://docs.sqlalchemy.org/en/14/orm/cascades.html#passive-deletes)) для получения более подробной информации об этом.

Еще раз обратите внимание, что это поведение относится **только** к опции [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary), используемой с отношением [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship). При работе с ассоциативными таблицами, которые отображены явно и не представлены в опции [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary) соответствующего отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), вместо этого можно использовать каскадные правила для автоматического удаления объектов в ответ на удаление связанного объекта — см. Каскады ([Cascades](https://docs.sqlalchemy.org/en/14/orm/cascades.html#unitofwork-cascades)) для получения информации о эта особенность.

{% hint style="info" %}
Смотри также:

[Using delete cascade with many-to-many relationships](https://docs.sqlalchemy.org/en/14/orm/cascades.html#cascade-delete-many-to-many)

[Using foreign key ON DELETE with many-to-many relationships](https://docs.sqlalchemy.org/en/14/orm/cascades.html#passive-deletes-many-to-many)
{% endhint %}

## Ассоциативный объект

Шаблон ассоциативного объекта — это вариант «многие ко многим»: он используется, когда ваша ассоциативная таблица содержит дополнительные столбцы помимо тех, которые являются внешними ключами для левой и правой таблиц. Вместо использования аргумента [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary) вы сопоставляете новый класс непосредственно с таблицей ассоциаций. Левая сторона отношения ссылается на объект ассоциации по принципу «один ко многим», а класс ассоциации ссылается на правую сторону по принципу «многие к одному». Ниже мы иллюстрируем таблицу ассоциаций, сопоставленную с классом **Association**, которая включает столбец с именем **extra\_data**, представляющий собой строковое значение, которое хранится вместе с каждой ассоциацией между **Parent** и **Child**:

```python
class Association(Base):
    __tablename__ = 'association'
    left_id = Column(ForeignKey('left.id'), primary_key=True)
    right_id = Column(ForeignKey('right.id'), primary_key=True)
    extra_data = Column(String(50))
    child = relationship("Child")

class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)
    children = relationship("Association")

class Child(Base):
    __tablename__ = 'right'
    id = Column(Integer, primary_key=True)
```

Как всегда, в двунаправленной версии используются [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) или [relationship.backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref):

```python
class Association(Base):
    __tablename__ = 'association'
    left_id = Column(ForeignKey('left.id'), primary_key=True)
    right_id = Column(ForeignKey('right.id'), primary_key=True)
    extra_data = Column(String(50))
    child = relationship("Child", back_populates="parents")
    parent = relationship("Parent", back_populates="children")

class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)
    children = relationship("Association", back_populates="parent")

class Child(Base):
    __tablename__ = 'right'
    id = Column(Integer, primary_key=True)
    parents = relationship("Association", back_populates="child")
```

Работа с шаблоном ассоциации в его прямой форме требует, чтобы дочерние объекты были связаны с экземпляром ассоциации до того, как они будут присоединены к родителю; аналогично доступ от родителя к дочернему идет через объект ассоциации:

```python
# создать родителя, добавить дочерний элемент через ассоциацию
p = Parent()
a = Association(extra_data="some data")
a.child = Child()
p.children.append(a)

# перебирать дочерние объекты через ассоциацию,
# включая атрибуты ассоциации
for assoc in p.children:
    print(assoc.extra_data)
    print(assoc.child)
```

Чтобы улучшить шаблон объекта ассоциации, чтобы прямой доступ к объекту ассоциации **Association** был необязательным, SQLAlchemy предоставляет расширение прокси ассоциации ([Association Proxy](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html)). Это расширение позволяет настраивать атрибуты, которые будут иметь доступ к двум «переходам» с одним доступом, один «переход» к связанному объекту и второй к целевому атрибуту.

{% hint style="danger" %}
Шаблон ассоциативного объекта **не координирует изменения с отдельным отношением, которое отображает ассоциативную таблицу как «вторичную»**.

Ниже изменения, внесенные в **Parent.children**, не будут согласовываться с изменениями, внесенными в **Parent.child\_associations** или **Child.parent\_associations** в Python; в то время как все эти отношения будут продолжать нормально функционировать сами по себе, изменения в одном не будут отображаться в другом, пока не истечет срок действия сеанса [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), что обычно происходит автоматически после [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit):

```python
class Association(Base):
    __tablename__ = 'association'

    left_id = Column(ForeignKey('left.id'), primary_key=True)
    right_id = Column(ForeignKey('right.id'), primary_key=True)
    extra_data = Column(String(50))

    child = relationship("Child", backref="parent_associations")
    parent = relationship("Parent", backref="child_associations")

class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)

    children = relationship("Child", secondary="association")

class Child(Base):
    __tablename__ = 'right'
    id = Column(Integer, primary_key=True)
```

Кроме того, точно так же, как изменения в одном отношении не отражаются автоматически в других, запись одних и тех же данных в оба отношения также вызовет конфликты операторов **INSERT** или **DELETE**, например, ниже, где мы дважды устанавливаем одно и то же отношение между родительским и дочерним объектами. :

```python
p1 = Parent()
c1 = Child()
p1.children.append(c1)

# избыточный, приведет к дублированию INSERT в ассоциации
p1.child_associations.append(Association(child=c1))
```

Если вы знаете, что делаете, можно использовать отображение, подобное приведенному выше, хотя может быть хорошей идеей применить параметр **`viewonly=True`** к «вторичному» отношению, чтобы избежать проблемы регистрации избыточных изменений. Однако, чтобы получить надежный шаблон, который допускает простую связь между двумя объектами **Parent->Child**, все еще используя шаблон объекта ассоциации, используйте расширение прокси-сервера ассоциации, как описано в прокси-сервере ассоциации ([Association Proxy](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html)).
{% endhint %}
