# Шаблоны сохранения особых отношений

## Строки, которые указывают сами на себя/Взаимозависимые строки

Это очень специфический случай, когда отношение **relationship()** должно выполнять **INSERT** и второе **UPDATE** для правильного заполнения строки (и наоборот **UPDATE** и **DELETE** для удаления без нарушения ограничений внешнего ключа). Два варианта использования:

* Таблица содержит внешний ключ для самой себя, и одна строка будет иметь значение внешнего ключа, указывающее на ее собственный первичный ключ.
* Каждая из двух таблиц содержит внешний ключ, ссылающийся на другую таблицу, причем строка в каждой таблице ссылается на другую.

Например:

```bash
          user
---------------------------------
user_id    name   related_user_id
   1       'ed'          1
```

Или:

```bash
             widget                                                  entry
-------------------------------------------             ---------------------------------
widget_id     name        favorite_entry_id             entry_id      name      widget_id
   1       'somewidget'          5                         5       'someentry'     1
```

В первом случае строка указывает сама на себя. Технически база данных, использующая последовательности, такие как PostgreSQL или Oracle, может сразу **INSERT** строку, используя ранее сгенерированное значение, но базы данных, которые полагаются на идентификаторы первичного ключа в стиле автоинкремента, не могут. Отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) всегда предполагает «родительско-дочернюю» модель заполнения строк во время сброса, поэтому, если вы не заполняете столбцы первичного/внешнего ключа напрямую, отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) должно использовать два оператора.

Во втором случае строка «**widget**» должна быть вставлена ​​перед любыми ссылающимися строками «**entry**», но тогда столбец «**favorite\_entry\_id**» этой строки «**widget**» не может быть установлен до тех пор, пока не будут сгенерированы строки «**entry**». В этом случае обычно невозможно вставить строки «**widget**» и «**entry**», используя всего два оператора **INSERT**; **UPDATE** должно быть выполнено, чтобы сохранить ограничения внешнего ключа. Исключение составляют случаи, когда внешние ключи настроены как «отложенные до фиксации» - _"deferred until commit"_ (функция, поддерживаемая некоторыми базами данных) и если идентификаторы заполнялись вручную (опять же, по сути, в обход отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship)).

Чтобы разрешить использование дополнительного оператора **UPDATE**, мы используем параметр [relationship.post\_update](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.post\_update) для отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship). Это указывает, что связь между двумя строками должна быть создана с помощью оператора **UPDATE** после того, как обе строки были **INSERTED**; это также вызывает деассоциацию строк друг с другом через **UPDATE** до того, как будет выдано **DELETE**. Флаг должен быть размещен только на одном из отношений, предпочтительно на стороне многие-к-одному. Ниже мы проиллюстрируем полный пример, включающий две конструкции [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey):

```python
from sqlalchemy import Integer, ForeignKey, Column
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class Entry(Base):
    __tablename__ = 'entry'
    entry_id = Column(Integer, primary_key=True)
    widget_id = Column(Integer, ForeignKey('widget.widget_id'))
    name = Column(String(50))

class Widget(Base):
    __tablename__ = 'widget'

    widget_id = Column(Integer, primary_key=True)
    favorite_entry_id = Column(Integer,
                            ForeignKey('entry.entry_id',
                            name="fk_favorite_entry"))
    name = Column(String(50))

    entries = relationship(Entry, primaryjoin=
                                    widget_id==Entry.widget_id)
    favorite_entry = relationship(Entry,
                                primaryjoin=
                                    favorite_entry_id==Entry.entry_id,
                                post_update=True)
```

Когда структура в соответствии с приведенной выше конфигурацией сбрасывается (flushed), строка «**widget**» будет **INSERTed** за вычетом значения «**favorite\_entry\_id**», затем все строки «**entry**» будут **INSERTed**, ссылаясь на родительскую строку «**widget**», а затем будет заполнен оператором **UPDATE** столбец «**favorite\_entry\_id**» таблицы «**widget**» (пока что это одна строка за раз):

```python
>>> w1 = Widget(name='somewidget')
>>> e1 = Entry(name='someentry')
>>> w1.favorite_entry = e1
>>> w1.entries = [e1]
>>> session.add_all([w1, e1])
>>> session.commit()
```

```sql
BEGIN (implicit)
INSERT INTO widget (favorite_entry_id, name) VALUES (?, ?)
(None, 'somewidget')
INSERT INTO entry (widget_id, name) VALUES (?, ?)
(1, 'someentry')
UPDATE widget SET favorite_entry_id=? WHERE widget.widget_id = ?
(1, 1)
COMMIT
```

Дополнительная конфигурация, которую мы можем указать, заключается в предоставлении более полного ограничения внешнего ключа для **Widget**, чтобы гарантировалось, что **favorite\_entry\_id** ссылается на **Entry**, которая также ссылается на этот **Widget**. Мы можем использовать составной внешний ключ, как показано ниже:

```python
from sqlalchemy import Integer, ForeignKey, String, \
        Column, UniqueConstraint, ForeignKeyConstraint
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class Entry(Base):
    __tablename__ = 'entry'
    entry_id = Column(Integer, primary_key=True)
    widget_id = Column(Integer, ForeignKey('widget.widget_id'))
    name = Column(String(50))
    __table_args__ = (
        UniqueConstraint("entry_id", "widget_id"),
    )

class Widget(Base):
    __tablename__ = 'widget'

    widget_id = Column(Integer, autoincrement='ignore_fk', primary_key=True)
    favorite_entry_id = Column(Integer)

    name = Column(String(50))

    __table_args__ = (
        ForeignKeyConstraint(
            ["widget_id", "favorite_entry_id"],
            ["entry.widget_id", "entry.entry_id"],
            name="fk_favorite_entry"
        ),
    )

    entries = relationship(Entry, primaryjoin=
                                    widget_id==Entry.widget_id,
                                    foreign_keys=Entry.widget_id)
    favorite_entry = relationship(Entry,
                                primaryjoin=
                                    favorite_entry_id==Entry.entry_id,
                                foreign_keys=favorite_entry_id,
                                post_update=True)
```

Вышеупомянутое сопоставление имеет составное [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint), соединяющее столбцы **widget\_id** и **favourite\_entry\_id**. Чтобы убедиться, что **Widget.widget\_id** остается «автоинкрементным» столбцом, мы указываем [Column.autoincrement](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.autoincrement) значение «**ignore\_fk**» в столбце [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), и дополнительно для каждого отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) мы должны ограничить те столбцы, которые рассматриваются как часть внешнего ключа для целей присоединения и перекрестной популяции.

## Изменяемые первичные ключи/каскады обновлений

Когда первичный ключ объекта изменяется, связанные элементы, которые ссылаются на первичный ключ, также должны быть обновлены. Для баз данных, обеспечивающих ссылочную целостность, наилучшей стратегией является использование функции базы данных **ON UPDATE CASCADE** для распространения изменений первичного ключа на внешние ключи, на которые ссылаются, — значения не могут быть рассинхронизированы в любой момент, если только ограничения не помечены как «отложенные» (deferrable) , то есть не применяется до завершения транзакции.

**Настоятельно рекомендуется**, чтобы приложение, которое стремится использовать естественные первичные ключи с изменяемыми значениями, использовало возможности базы данных **ON UPDATE CASCADE**. Пример сопоставления, который иллюстрирует это:

```python
class User(Base):
    __tablename__ = 'user'
    __table_args__ = {'mysql_engine': 'InnoDB'}

    username = Column(String(50), primary_key=True)
    fullname = Column(String(100))

    addresses = relationship("Address")


class Address(Base):
    __tablename__ = 'address'
    __table_args__ = {'mysql_engine': 'InnoDB'}

    email = Column(String(50), primary_key=True)
    username = Column(String(50),
                ForeignKey('user.username', onupdate="cascade")
            )
```

Выше мы иллюстрируем **onupdate="cascade"** для объекта [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey), а также иллюстрируем настройку **mysql\_engine='InnoDB'**, которая на серверной части MySQL обеспечивает использование механизма **InnoDB**, поддерживающего ссылочную целостность. При использовании SQLite необходимо включить ссылочную целостность, используя конфигурацию, описанную в разделе Поддержка внешних ключей ([Foreign Key Support](https://docs.sqlalchemy.org/en/14/dialects/sqlite.html#sqlite-foreign-keys)).

{% hint style="info" %}
Смотри также:

[Using foreign key ON DELETE cascade with ORM relationships](https://docs.sqlalchemy.org/en/14/orm/cascades.html#passive-deletes) - поддержка ON DELETE CASCADE с отношениями relationships

[`mapper.passive_updates`](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.passive\_updates) - особенности, похожие на [`mapper()`](../konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/func-mapper.md#function-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary)``
{% endhint %}

### Имитация ограниченного ON UPDATE CASCADE без поддержки внешнего ключа

В тех случаях, когда используется база данных, не поддерживающая ссылочную целостность, и используются естественные первичные ключи с изменяемыми значениями, SQLAlchemy предлагает функцию, позволяющую в **ограниченной степени** распространять значения первичного ключа на внешние ключи, на которые уже есть ссылки, путем отправки инструкции **UPDATE** для столбцов внешнего ключа, которые немедленно ссылаются на столбец первичного ключа, значение которого изменилось. Основными платформами без функций ссылочной целостности являются MySQL, когда используется механизм хранения MyISAM, и SQLite, когда прагма `PRAGMA foreign_keys=ON` не используется. База данных Oracle также не поддерживает **ON UPDATE CASCADE**, но, поскольку она по-прежнему обеспечивает ссылочную целостность, требует, чтобы ограничения были помечены как отложенные, чтобы SQLAlchemy мог генерировать операторы **UPDATE**.

Эта функция активируется установкой флага [relationship.passive\_updates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.passive\_updates) в значение `False`, что наиболее предпочтительно для отношения «один ко многим» или «многие ко многим» [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship). Когда «обновления» больше не являются «пассивными», это указывает на то, что SQLAlchemy будет выдавать операторы **UPDATE** индивидуально для объектов, на которые ссылается коллекция, на которую ссылается родительский объект с изменяющимся значением первичного ключа. Это также означает, что коллекции будут полностью загружены в память, если они еще не присутствуют локально.

Наше предыдущее сопоставление с использованием **passive\_updates=False** выглядит так:

```python
class User(Base):
    __tablename__ = 'user'

    username = Column(String(50), primary_key=True)
    fullname = Column(String(100))

    # passive_updates=False необходимо *только*, если база данных
    # не имеет возможности использовать ON UPDATE CASCADE
    addresses = relationship("Address", passive_updates=False)

class Address(Base):
    __tablename__ = 'address'

    email = Column(String(50), primary_key=True)
    username = Column(String(50), ForeignKey('user.username'))
```

Основные ограничения для **passive\_updates=False** включают:

* он работает намного хуже, чем прямая функция базы данных **ON UPDATE CASCADE**, потому что ему необходимо полностью предварительно загрузить затронутые коллекции с помощью **SELECT**, а также выдать операторы **UPDATE** для этих значений, которые он будет пытаться запускать «пакетами», но по-прежнему будет работать на каждой основе строки на уровне DBAPI.
* функция не может «**cascade**» более чем на один уровень. То есть, если сопоставление **X** имеет внешний ключ, который ссылается на первичный ключ сопоставления **Y**, но если сопоставление первичного ключа **Y** само по себе является внешним ключом для сопоставления **Z**, `passive_updates=False` не может каскадировать изменение значения первичного ключа с **Z** на **X**.
* настройка **passive\_updates=False** только на стороне отношения «многие к одному» не будет иметь полного эффекта, поскольку единица работы ищет только текущую карту идентификаторов для объектов, которые могут ссылаться на объект с изменяющимся первичным ключом, а не по всей базе.

Поскольку практически все базы данных, кроме Oracle, теперь поддерживают **ON UPDATE CASCADE**, настоятельно рекомендуется использовать традиционную поддержку **ON UPDATE CASCADE** в случае использования естественных и изменяемых значений первичного ключа.
