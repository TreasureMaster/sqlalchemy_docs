# Наследование объединенной таблицы

## Наследование объединенной таблицы

При наследовании объединенных таблиц каждый класс в иерархии классов представлен отдельной таблицей. Запрос определенного подкласса в иерархии будет отображаться как SQL **JOIN** для всех таблиц в его пути наследования. Если запрошенный класс является базовым классом, **по умолчанию в оператор SELECT включается только базовая таблица**. Во всех случаях окончательный класс для создания экземпляра для данной строки определяется столбцом дискриминатора или выражением, которое работает с базовой таблицей. Когда подкласс загружается **только** для базовой таблицы, у результирующих объектов сначала будут заполнены базовые атрибуты; атрибуты, которые являются локальными для подкласса, будут [лениво загружаться](https://docs.sqlalchemy.org/en/14/glossary.html#term-lazy-load) при доступе к ним. В качестве альтернативы существуют параметры, которые могут изменить поведение по умолчанию, позволяя запросу включать столбцы, соответствующие нескольким таблицам/подклассам.

Базовый класс в объединенной иерархии наследования настраивается с дополнительными аргументами, которые будут ссылаться на столбец полиморфного дискриминатора, а также на идентификатор базового класса:

```python
class Employee(Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    type = Column(String(50))

    __mapper_args__ = {
        'polymorphic_identity':'employee',
        'polymorphic_on':type
    }
```

Выше установлен дополнительный тип столбца **type**, который действует как дискриминатор, настроенный как таковой с помощью параметра [mapper.polymorphic\_on](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.polymorphic\_on). В этом столбце будет храниться значение, указывающее тип объекта, представленного в строке. Столбец может иметь любой тип данных, хотя строковый и целочисленный являются наиболее распространенными. Фактическое значение данных, которое должно применяться к этому столбцу для конкретной строки в базе данных, указывается с помощью параметра [mapper.polymorphic\_identity](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.polymorphic\_identity), описанного ниже.

Хотя выражение полиморфного дискриминатора не является строго необходимым, оно требуется, если требуется полиморфная загрузка. Создание простого столбца в базовой таблице — самый простой способ добиться этого, однако очень сложные сопоставления наследования могут даже настроить выражение SQL, такое как оператор **CASE**, в качестве полиморфного дискриминатора.

{% hint style="info" %}
В настоящее время **для всей иерархии наследования можно настроить только один столбец дискриминатора или SQL-выражение**, обычно для самого базового класса в иерархии. «Каскадные» выражения полиморфного дискриминатора пока не поддерживаются.
{% endhint %}

Затем мы определяем подклассы **Engineer** и **Manager** класса **Employee**. Каждый содержит столбцы, представляющие атрибуты, уникальные для представляемого ими подкласса. Каждая таблица также должна содержать столбец (или столбцы) первичного ключа, а также ссылку внешнего ключа на родительскую таблицу:

```python
class Engineer(Employee):
    __tablename__ = 'engineer'
    id = Column(Integer, ForeignKey('employee.id'), primary_key=True)
    engineer_name = Column(String(30))

    __mapper_args__ = {
        'polymorphic_identity':'engineer',
    }

class Manager(Employee):
    __tablename__ = 'manager'
    id = Column(Integer, ForeignKey('employee.id'), primary_key=True)
    manager_name = Column(String(30))

    __mapper_args__ = {
        'polymorphic_identity':'manager',
    }
```

В приведенном выше примере каждое сопоставление указывает параметр [mapper.polymorphic\_identity](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.polymorphic\_identity) в своих аргументах сопоставления. Это значение заполняет столбец, указанный параметром [mapper.polymorphic\_on](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.polymorphic\_on), установленным в базовом преобразователе. Параметр [mapper.polymorphic\_identity](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.polymorphic\_identity) должен быть уникальным для каждого сопоставленного класса во всей иерархии, и для каждого сопоставленного класса должно быть только одно **«identity»**; как отмечалось выше, «каскадные» идентификаторы, когда некоторые подклассы вводят второй идентификатор, не поддерживаются.

ORM использует значение, установленное [mapper.polymorphic\_identity](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.polymorphic\_identity), чтобы определить, к какому классу принадлежит строка при полиморфной загрузке строк. В приведенном выше примере каждая строка, представляющая **Employes**, будет иметь значение **«employee»** в строке типа; аналогично, каждый **Engineer** получит значение **«engineer»**, а каждый **Manager** получит значение **«manager»**. Независимо от того, использует ли сопоставление наследования отдельные соединенные таблицы для подклассов, как при наследовании объединенных таблиц, или всю одну таблицу, как при наследовании одной таблицы, ожидается, что это значение будет сохранено и доступно для ORM при запросе. Параметр [mapper.polymorphic\_identity](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.polymorphic\_identity) также применяется к наследованию конкретной таблицы, но фактически не сохраняется; подробности см. в следующем разделе «[Наследование конкретных таблиц](nasledovanie-konkretnoi-tablicy.md)».

В полиморфной конфигурации чаще всего ограничение внешнего ключа устанавливается для того же столбца или столбцов, что и сам первичный ключ, однако это не требуется; столбец, отличный от первичного ключа, также может ссылаться на родителя через внешний ключ. Способ построения **JOIN** от базовой таблицы к подклассам также можно напрямую настроить, однако это редко требуется.

{% hint style="info" %}
**Объединенные первичные ключи наследования**

Одним из естественных эффектов конфигурации наследования объединенных таблиц является то, что идентификатор любого отображаемого объекта можно полностью определить только по строкам в базовой таблице. Это имеет очевидные преимущества, поэтому SQLAlchemy всегда считает столбцы первичного ключа объединенного класса наследования только столбцами базовой таблицы. Другими словами, столбцы идентификаторов **id** таблиц **engineer** и **manager** не используются для поиска объектов **Engineer** или **Manager** — учитывается только значение в `employee.id`. `engineer.id` и `manager.id`, конечно, по-прежнему важны для правильной работы шаблона в целом, поскольку они используются для поиска присоединяемой строки после того, как родительская строка была определена в операторе.
{% endhint %}

Когда сопоставление объединенного наследования завершено, запрос к **Employee** вернет комбинацию объектов **Employee**, **Engineer** и **Manager**. Недавно сохраненные объекты **Engineer**, **Manager** и **Employee** будут автоматически заполнять столбец `employee.type` правильным значением «дискриминатора», в данном случае **«engineer»**, **«manager»** или **«employee»**, в зависимости от ситуации.

### Отношения с объединенным наследованием

Отношения полностью поддерживаются наследованием объединенных таблиц. Отношение, включающее класс объединенного наследования, должно быть нацелено на класс в иерархии, который также соответствует ограничению внешнего ключа; ниже, поскольку таблица **employee** имеет ограничение внешнего ключа обратно в таблицу **company**, отношения устанавливаются между **Company** и **Employee**:

```python
class Company(Base):
    __tablename__ = 'company'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    employees = relationship("Employee", back_populates="company")

class Employee(Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    type = Column(String(50))
    company_id = Column(ForeignKey('company.id'))
    company = relationship("Company", back_populates="employees")

    __mapper_args__ = {
        'polymorphic_identity':'employee',
        'polymorphic_on':type
    }

class Manager(Employee):
    # ...

class Engineer(Employee):
    # ...
```

Если ограничение внешнего ключа относится к таблице, соответствующей подклассу, отношение должно быть нацелено на этот подкласс. В приведенном ниже примере существует ограничение внешнего ключа от **manager** к **company**, поэтому отношения устанавливаются между классами **Manager** и **Company**:

```python
class Company(Base):
    __tablename__ = 'company'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    managers = relationship("Manager", back_populates="company")

class Employee(Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    type = Column(String(50))

    __mapper_args__ = {
        'polymorphic_identity':'employee',
        'polymorphic_on':type
    }

class Manager(Employee):
    __tablename__ = 'manager'
    id = Column(Integer, ForeignKey('employee.id'), primary_key=True)
    manager_name = Column(String(30))

    company_id = Column(ForeignKey('company.id'))
    company = relationship("Company", back_populates="managers")

    __mapper_args__ = {
        'polymorphic_identity':'manager',
    }

class Engineer(Employee):
    # ...
```

Выше класс **Manager** будет иметь атрибут `Manager.company`; **Company** будет иметь атрибут `Company.managers`, который всегда загружается при объединении таблиц **employee** и **manager** вместе.

### Загрузка объединенных сопоставлений наследования

См. разделы «Загрузка иерархий наследования» ([Loading Inheritance Hierarchies](https://docs.sqlalchemy.org/en/14/orm/inheritance\_loading.html)) и «Загрузка объектов с наследованием объединенных таблиц» ([Loading objects with joined table inheritance](https://docs.sqlalchemy.org/en/14/orm/inheritance\_loading.html#loading-joined-inheritance)), чтобы узнать о методах загрузки наследования, включая настройку таблиц, которые необходимо запрашивать как во время настройки преобразователя mapper, так и во время запроса.
