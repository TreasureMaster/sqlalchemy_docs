# Составные типы колонок

## Составные типы колонок

Наборы столбцов могут быть связаны с одним пользовательским типом данных. ORM предоставляет единственный атрибут, который представляет группу столбцов с использованием предоставленного вами класса.

Простой пример представляет пары столбцов как объект **Point**. **Point** представляет такую пару, как `.x` и `.y`:

```python
class Point(object):
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __composite_values__(self):
        return self.x, self.y

    def __repr__(self):
        return "Point(x=%r, y=%r)" % (self.x, self.y)

    def __eq__(self, other):
        return isinstance(other, Point) and \
            other.x == self.x and \
            other.y == self.y

    def __ne__(self, other):
        return not self.__eq__(other)
```

Требования к классу пользовательского типа данных заключаются в том, что он должен иметь конструктор, который принимает позиционные аргументы, соответствующие формату его столбца, а также предоставляет метод **\_\_composite\_values\_\_()**, который возвращает состояние объекта в виде списка или кортежа в порядке его основанных на столбце атрибутах. Он также должен предоставлять адекватные методы **\_\_eq\_\_()** и **\_\_ne\_\_()**, которые проверяют равенство двух экземпляров.

Мы создадим сопоставление с вершинами таблицы **vertices**, которая представляет две точки как `x1/y1` и `x2/y2`. Обычно они создаются как объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column). Затем функция [composite()](https://docs.sqlalchemy.org/en/14/orm/composites.html#sqlalchemy.orm.composite) используется для назначения новых атрибутов, которые будут представлять наборы столбцов через класс **Point**:

```python
from sqlalchemy import Column, Integer
from sqlalchemy.orm import composite
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Vertex(Base):
    __tablename__ = 'vertices'

    id = Column(Integer, primary_key=True)
    x1 = Column(Integer)
    y1 = Column(Integer)
    x2 = Column(Integer)
    y2 = Column(Integer)

    start = composite(Point, x1, y1)
    end = composite(Point, x2, y2)
```

Приведенное выше классическое сопоставление определило бы каждый [composite()](https://docs.sqlalchemy.org/en/14/orm/composites.html#sqlalchemy.orm.composite) для существующей таблицы:

```python
mapper_registry.map_imperatively(Vertex, vertices_table, properties={
    'start':composite(Point, vertices_table.c.x1, vertices_table.c.y1),
    'end':composite(Point, vertices_table.c.x2, vertices_table.c.y2),
})
```

Теперь мы можем сохранять и использовать экземпляры **Vertex**, а также запрашивать их, используя атрибуты `.start` и `.end` для специальных экземпляров **Point**:

```python
>>> v = Vertex(start=Point(3, 4), end=Point(5, 6))
>>> session.add(v)
>>> q = session.query(Vertex).filter(Vertex.start == Point(3, 4))

>>> print(q.first().start)
Point(x=3, y=4)
```

```sql
BEGIN (implicit)
INSERT INTO vertices (x1, y1, x2, y2) VALUES (?, ?, ?, ?)
(3, 4, 5, 6)
SELECT vertices.id AS vertices_id,
        vertices.x1 AS vertices_x1,
        vertices.y1 AS vertices_y1,
        vertices.x2 AS vertices_x2,
        vertices.y2 AS vertices_y2
FROM vertices
WHERE vertices.x1 = ? AND vertices.y1 = ?
 LIMIT ? OFFSET ?
(3, 4, 1, 0)
```

| Название объекта                                                                                                          | Описание                                                                    |
| ------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [composite](https://docs.sqlalchemy.org/en/14/orm/composites.html#sqlalchemy.orm.composite)(class\_, \*attrs, \*\*kwargs) | Возвращает составное свойство на основе столбца для использования с Mapper. |

### _function_ sqlalchemy.orm.composite(_class\__, _\*attrs_, _\*\*kwargs_)

Возвращает составное свойство на основе столбца для использования с Mapper.

Полный пример использования см. в разделе документации по сопоставлению [Composite Column Types](sostavnye-tipy-kolonok.md).

[MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty), возвращаемый [composite()](sostavnye-tipy-kolonok.md#function-sqlalchemy.orm.composite-class\_-attrs-kwargs), является [CompositeProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.CompositeProperty).

#### Параметры:

* **class\_** - Класс «составного типа» или любой метод класса или вызываемый объект, который создаст новый экземпляр составного объекта с заданными значениями столбца по порядку.
* **\*cols** - Список объектов **Column** для сопоставления.
* **active\_history=False** - Значение `True` указывает, что «предыдущее» значение для скалярного атрибута должно быть загружено при замене, если оно еще не загружено. См. тот же флаг в [column\_property()](otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs).
* **group** - Имя группы для этого свойства, если оно помечено как отложенное.
* **deferred** - Когда значение равно `True`, свойство столбца является «отложенным», что означает, что оно не загружается сразу, а вместо этого загружается при первом доступе к атрибуту в экземпляре. См. также [deferred()](https://docs.sqlalchemy.org/en/14/orm/loading\_columns.html#sqlalchemy.orm.deferred).
* **comparator\_factory** - класс, который расширяет [Comparator](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.CompositeProperty.Comparator), который обеспечивает генерацию пользовательского предложения SQL для операций сравнения.
* **doc** - необязательная строка, которая будет применяться в качестве документа для дескриптора, привязанного к классу.
* **info** - Необязательный словарь данных, который будет заполнен атрибутом [MapperProperty.info](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty.info) этого объекта.

## Отслеживание мутаций на месте в композитах

Изменения существующего составного значения на месте не отслеживаются автоматически. Вместо этого составной класс должен явно предоставлять события своему родительскому объекту. Эта задача в значительной степени автоматизирована с помощью миксина [MutableComposite](https://docs.sqlalchemy.org/en/14/orm/extensions/mutable.html#sqlalchemy.ext.mutable.MutableComposite), который использует события для связывания каждого определяемого пользователем составного объекта со всеми родительскими ассоциациями. См. пример в разделе «Установление изменчивости композитов» ([Establishing Mutability on Composites](https://docs.sqlalchemy.org/en/14/orm/extensions/mutable.html#mutable-composites)).

## Переопределение операций сравнения для композитов

Операция сравнения **«равно»** по умолчанию производит **AND** всех соответствующих столбцов, приравненных друг к другу. Это можно изменить с помощью аргумента `comparator_factory` в [composite()](sostavnye-tipy-kolonok.md#function-sqlalchemy.orm.composite-class\_-attrs-kwargs), где мы указываем собственный класс [Comparator](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.CompositeProperty.Comparator) для определения существующих или новых операций. Ниже мы иллюстрируем оператор «больше чем», реализующий то же выражение, что и базовое «больше чем»:

```python
from sqlalchemy.orm.properties import CompositeProperty
from sqlalchemy import sql

class PointComparator(CompositeProperty.Comparator):
    def __gt__(self, other):
        """redefine the 'greater than' operation"""

        return sql.and_(*[a>b for a, b in
                          zip(self.__clause_element__().clauses,
                              other.__composite_values__())])

class Vertex(Base):
    ___tablename__ = 'vertices'

    id = Column(Integer, primary_key=True)
    x1 = Column(Integer)
    y1 = Column(Integer)
    x2 = Column(Integer)
    y2 = Column(Integer)

    start = composite(Point, x1, y1,
                        comparator_factory=PointComparator)
    end = composite(Point, x2, y2,
                        comparator_factory=PointComparator)
```

## Вложенные композиты

Составные объекты могут быть определены для работы в простых вложенных схемах путем переопределения поведения в составном классе для работы по желанию, а затем обычного сопоставления составного класса с полной длиной отдельных столбцов. Как правило, удобно определять отдельные конструкторы для пользовательского использования и использования для генерации из строки. Ниже мы реорганизуем класс **Vertex** в составной объект, который затем сопоставляется с классом **HasVertex**:

```python
from sqlalchemy.orm import composite

class Point(object):
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __composite_values__(self):
        return self.x, self.y

    def __repr__(self):
        return "Point(x=%r, y=%r)" % (self.x, self.y)

    def __eq__(self, other):
        return isinstance(other, Point) and \
            other.x == self.x and \
            other.y == self.y

    def __ne__(self, other):
        return not self.__eq__(other)

class Vertex(object):
    def __init__(self, start, end):
        self.start = start
        self.end = end

    @classmethod
    def _generate(self, x1, y1, x2, y2):
        """generate a Vertex from a row"""
        return Vertex(
            Point(x1, y1),
            Point(x2, y2)
        )

    def __composite_values__(self):
        return \
            self.start.__composite_values__() + \
            self.end.__composite_values__()

class HasVertex(Base):
    __tablename__ = 'has_vertex'
    id = Column(Integer, primary_key=True)
    x1 = Column(Integer)
    y1 = Column(Integer)
    x2 = Column(Integer)
    y2 = Column(Integer)

    vertex = composite(Vertex._generate, x1, y1, x2, y2)
```

Затем мы можем использовать приведенное выше отображение как:

```python
hv = HasVertex(vertex=Vertex(Point(1, 2), Point(3, 4)))

s.add(hv)
s.commit()

hv = s.query(HasVertex).filter(
    HasVertex.vertex == Vertex(Point(1, 2), Point(3, 4))).first()
print(hv.vertex.start)
print(hv.vertex.end)
```
