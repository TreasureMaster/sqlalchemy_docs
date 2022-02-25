# Отношения списка смежности

## Отношения списка смежности

Шаблон **списка смежности** является распространенным реляционным шаблоном, в котором таблица содержит ссылку внешнего ключа на себя, другими словами, это **самореферентное отношение**. Это наиболее распространенный способ представления иерархических данных в плоских таблицах. Другие методы включают **вложенные наборы**, иногда называемые «модифицированным предварительным порядком», а также **материализованный путь**. Несмотря на привлекательность модифицированного предварительного порядка при оценке его беглости в SQL-запросах, модель списка смежности, вероятно, является наиболее подходящим шаблоном для подавляющего большинства потребностей иерархического хранения по причинам параллелизма, снижения сложности, и что модифицированный предварительный порядок не имеет большого преимущества над приложением, которое может полностью загружать поддеревья в пространство приложения.

{% hint style="info" %}
Смотри также:

В этом разделе подробно описывается однотабличная версия самореферентного отношения. Для самореферентной связи, которая использует вторую таблицу в качестве ассоциативной таблицы, см. раздел Самореферентная связь «многие ко многим» ([Self-Referential Many-to-Many Relationship](https://docs.sqlalchemy.org/en/14/orm/join\_conditions.html#self-referential-many-to-many)).
{% endhint %}

В этом примере мы будем работать с одним сопоставленным классом с именем **Node**, представляющим древовидную структуру:

```python
class Node(Base):
    __tablename__ = 'node'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('node.id'))
    data = Column(String(50))
    children = relationship("Node")
```

При такой структуре график выглядит следующим образом:

```bash
root --+---> child1
       +---> child2 --+--> subchild1
       |              +--> subchild2
       +---> child3
```

Будет представлен такими данными, как:

```bash
id       parent_id     data
---      -------       ----
1        NULL          root
2        1             child1
3        1             child2
4        3             subchild1
5        3             subchild2
6        1             child3
```

Конфигурация отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) здесь работает так же, как и «нормальное» отношение «один ко многим», за исключением того, что «направление», которое может являться отношением «один ко многим» или «многие к одному», предполагается по умолчанию как "один ко многим". Чтобы установить отношение «многие к одному», добавляется дополнительная директива, известная как [relationship.remote\_side](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.remote\_side), которая представляет собой столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) или набор объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), указывающих на то, что следует считать «удаленными»:

```python
class Node(Base):
    __tablename__ = 'node'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('node.id'))
    data = Column(String(50))
    parent = relationship("Node", remote_side=[id])
```

Там, где выше, столбец _**id**_ применяется в качестве отношения [relationship.remote\_side](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.remote\_side) родительского отношения _**parent**_ [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), таким образом, устанавливается _**parent\_id**_ как «локальная» сторона, и тогда отношение ведет себя как «многие к одному».

Как всегда, оба направления могут быть объединены в двунаправленную связь с помощью функции [backref()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.backref):

```python
class Node(Base):
    __tablename__ = 'node'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('node.id'))
    data = Column(String(50))
    children = relationship("Node",
                backref=backref('parent', remote_side=[id])
            )
```

В состав SQLAlchemy включено несколько примеров, иллюстрирующих самореферентные стратегии; к ним относятся список смежности ([Adjacency List](https://docs.sqlalchemy.org/en/14/orm/examples.html#examples-adjacencylist)) и сохраняемость XML ([XML Persistence](https://docs.sqlalchemy.org/en/14/orm/examples.html#examples-xmlpersistence)).

### Составные списки смежности

Подкатегория отношения списка смежности — это редкий случай, когда конкретный столбец присутствует как на «локальной», так и на «удаленной» стороне условия соединения. Примером является класс **Folder** ниже; используя составной первичный ключ, столбец **account\_id** ссылается на себя, чтобы указать подпапки, которые находятся в той же учетной записи, что и родительская; в то время как **folder\_id** относится к определенной папке в этой учетной записи:

```python
class Folder(Base):
    __tablename__ = 'folder'
    __table_args__ = (
      ForeignKeyConstraint(
          ['account_id', 'parent_id'],
          ['folder.account_id', 'folder.folder_id']),
    )

    account_id = Column(Integer, primary_key=True)
    folder_id = Column(Integer, primary_key=True)
    parent_id = Column(Integer)
    name = Column(String)

    parent_folder = relationship("Folder",
                        backref="child_folders",
                        remote_side=[account_id, folder_id]
                  )
```

Выше мы передаем **account\_id** в список [relationship.remote\_side](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.remote\_side). Отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) распознает, что столбец **account\_id** здесь находится на обеих сторонах, и выравнивает «удаленный» столбец вместе со столбцом **folder\_id**, который он распознает как уникально присутствующий на «удаленной» стороне.

### Стратегии самореферентных запросов

Запрос самореферентных структур работает так же, как и любой другой запрос:

```python
# получить все ноды с именем 'child2'
session.query(Node).filter(Node.data=='child2')
```

Однако требуется особая осторожность при попытке соединения по внешнему ключу с одного уровня дерева на другой. В SQL для соединения таблицы с самой собой требуется, чтобы по крайней мере одна сторона выражения была «с псевдонимом» (aliased), чтобы на нее можно было однозначно ссылаться.

Как вы помните из статьи «[Использование псевдонимов](../uchebnoe-posobie-po-relyacionnym-obektam-1.x-api/tutorial-chast-3-relationship-join.md#ispolzovanie-psevdonimov-aliases)» в руководстве по ORM, конструкция [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) обычно используется для предоставления «псевдонима» объекта ORM. Присоединение от узла **Node** к самому себе с использованием этой техники выглядит так:

```python
from sqlalchemy.orm import aliased

nodealias = aliased(Node)
session.query(Node).filter(Node.data=='subchild1').\
                join(Node.parent.of_type(nodealias)).\
                filter(nodealias.data=="child2").\
                all()
```

```sql
SELECT node.id AS node_id,
        node.parent_id AS node_parent_id,
        node.data AS node_data
FROM node JOIN node AS node_1
    ON node.parent_id = node_1.id
WHERE node.data = ?
    AND node_1.data = ?
['subchild1', 'child2']
```

Пример использования [aliased()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.aliased) для объединения произвольно длинной цепочки самоссылающихся узлов см. в разделе [XML Persistence](https://docs.sqlalchemy.org/en/14/orm/examples.html#examples-xmlpersistence).

### Настройка самореферентной жадной загрузки

Жадная загрузка отношений происходит с использованием соединений или внешних соединений из родительской таблицы в дочернюю во время обычной операции запроса, так что родительская и ее непосредственная дочерняя коллекция или ссылка могут быть заполнены из одного оператора SQL или второго оператора для всех непосредственных дочерних коллекций. Объединение SQLAlchemy и активная загрузка подзапросов используют таблицы с псевдонимами во всех случаях при присоединении к связанным элементам, поэтому они совместимы с самореферентным объединением. Однако, чтобы использовать нетерпеливую загрузку с самореферентным отношением, SQLAlchemy необходимо сообщить, на скольких уровнях он должен выполнять соединение и/или запрос; иначе жадной загрузки вообще не будет. Этот параметр глубины настраивается через **relationship.join\_depth**:

```python
class Node(Base):
    __tablename__ = 'node'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('node.id'))
    data = Column(String(50))
    children = relationship("Node",
                    lazy="joined",
                    join_depth=2)

session.query(Node).all()
```

```sql
SELECT node_1.id AS node_1_id,
        node_1.parent_id AS node_1_parent_id,
        node_1.data AS node_1_data,
        node_2.id AS node_2_id,
        node_2.parent_id AS node_2_parent_id,
        node_2.data AS node_2_data,
        node.id AS node_id,
        node.parent_id AS node_parent_id,
        node.data AS node_data
FROM node
    LEFT OUTER JOIN node AS node_2
        ON node.id = node_2.parent_id
    LEFT OUTER JOIN node AS node_1
        ON node_2.id = node_1.parent_id
[]
```
