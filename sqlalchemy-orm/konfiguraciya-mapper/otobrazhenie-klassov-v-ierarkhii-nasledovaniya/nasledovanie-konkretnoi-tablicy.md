# Наследование конкретной таблицы

## Наследование конкретной таблицы

Конкретное наследование сопоставляет каждый подкласс с собственной отдельной таблицей, каждая из которых содержит все столбцы, необходимые для создания экземпляра этого класса. Конкретная конфигурация наследования по умолчанию запрашивает неполиморфно; запрос для определенного класса будет запрашивать только таблицу этого класса и возвращать только экземпляры этого класса. Полиморфная загрузка конкретных классов включается путем настройки в преобразователе специального **SELECT**, который обычно создается как **UNION** всех таблиц.

{% hint style="danger" %}
Наследование конкретных таблиц **намного сложнее**, чем наследование объединенных или одиночных таблиц, и **гораздо более ограничено по функциональности**, особенно в отношении его использования со связями, быстрой загрузкой и полиморфной загрузкой. При полиморфном использовании он создает **очень большие запросы** с **UNION**, которые не будут выполняться так же хорошо, как простые соединения. Настоятельно рекомендуется, если требуется гибкость в загрузке отношений и полиморфной загрузке, по возможности использовать объединенное или одиночное наследование таблиц. Если полиморфная загрузка не требуется, то можно использовать простые сопоставления без наследования, если каждый класс полностью ссылается на свою собственную таблицу.
{% endhint %}

В то время как объединенное и однотабличное наследование хорошо работают с «полиморфной» загрузкой, с конкретным наследованием дело обстоит сложнее. По этой причине конкретное наследование более подходит, когда **не требуется полиморфная загрузка**. Установление отношений, включающих конкретные классы наследования, также более неудобно.

Чтобы класс использовал конкретное наследование, добавьте параметр [mapper.concrete](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.concrete) в файл **\_\_mapper\_args\_\_**. Это указывает как декларативному, так и сопоставлению, что таблицу суперкласса не следует рассматривать как часть сопоставления:

```python
class Employee(Base):
    __tablename__ = 'employee'

    id = Column(Integer, primary_key=True)
    name = Column(String(50))

class Manager(Employee):
    __tablename__ = 'manager'

    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    manager_data = Column(String(50))

    __mapper_args__ = {
        'concrete': True
    }

class Engineer(Employee):
    __tablename__ = 'engineer'

    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    engineer_info = Column(String(50))

    __mapper_args__ = {
        'concrete': True
    }
```

Следует отметить два критических момента:

* Мы **должны явно определить все столбцы** для каждого подкласса, даже для тех, которые имеют одно и то же имя. Такой столбец, как **Employee.name** здесь, **не копируется** в таблицы, сопоставленные **Manager** или **Engineer** для нас.
* в то время как классы **Engineer** и **Manager** отображаются в отношениях наследования с **Employee**, они по-прежнему **не включают полиморфную загрузку**. Это означает, что если мы запрашиваем объекты **Employee**, таблицы **Manager** и **Engineer** вообще не запрашиваются.

### Конфигурация конкретной полиморфной нагрузки

Полиморфная загрузка с конкретным наследованием требует, чтобы специализированный **SELECT** был настроен для каждого базового класса, который должен иметь полиморфную загрузку. Этот **SELECT** должен иметь возможность доступа ко всем сопоставленным таблицам по отдельности и обычно представляет собой оператор **UNION**, созданный с использованием вспомогательной функции SQLAlchemy [polymorphic\_union()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.polymorphic\_union).

Как обсуждалось в разделе Загрузка иерархий наследования ([Loading Inheritance Hierarchies](https://docs.sqlalchemy.org/en/14/orm/inheritance\_loading.html)), конфигурации наследования mapper любого типа можно настроить для загрузки из специального выбираемого по умолчанию с помощью аргумента [mapper.with\_polymorphic](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.with\_polymorphic). Текущий общедоступный API требует, чтобы этот аргумент был установлен в [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper) при его первом создании.

Однако в случае декларативного класса оба, и mapper, и сопоставляемая таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) создаются одновременно, в момент определения сопоставленного класса. Это означает, что аргумент [mapper.with\_polymorphic](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.with\_polymorphic) еще не может быть предоставлен, поскольку объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), соответствующие подклассам, еще не определены.

Существует несколько стратегий для разрешения этого цикла, однако Declarative предоставляет вспомогательные классы [ConcreteBase](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.ConcreteBase) и [AbstractConcreteBase](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.AbstractConcreteBase), которые решают эту проблему за кулисами.

Используя [ConcreteBase](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.ConcreteBase), мы можем настроить наше конкретное сопоставление почти так же, как мы делаем другие формы сопоставления наследования:

```python
from sqlalchemy.ext.declarative import ConcreteBase

class Employee(ConcreteBase, Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))

    __mapper_args__ = {
        'polymorphic_identity': 'employee',
        'concrete': True
    }

class Manager(Employee):
    __tablename__ = 'manager'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    manager_data = Column(String(40))

    __mapper_args__ = {
        'polymorphic_identity': 'manager',
        'concrete': True
    }

class Engineer(Employee):
    __tablename__ = 'engineer'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    engineer_info = Column(String(40))

    __mapper_args__ = {
        'polymorphic_identity': 'engineer',
        'concrete': True
    }
```

Выше Declarative устанавливает полиморфный выбор для класса **Employee** во время «инициализации» преобразователя; это этап поздней настройки для преобразователей, который разрешает другие зависимые преобразователи. Помощник [ConcreteBase](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.ConcreteBase) использует функцию [polymorphic\_union()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.polymorphic\_union) для создания **UNION** всех таблиц с конкретным отображением после того, как все остальные классы настроены, а затем настраивает этот оператор с уже существующим модулем отображения базового класса.

При выборе полиморфное объединение создает такой запрос:

```python
session.query(Employee).all()
```

```sql
SELECT
    pjoin.id AS pjoin_id,
    pjoin.name AS pjoin_name,
    pjoin.type AS pjoin_type,
    pjoin.manager_data AS pjoin_manager_data,
    pjoin.engineer_info AS pjoin_engineer_info
FROM (
    SELECT
        employee.id AS id,
        employee.name AS name,
        CAST(NULL AS VARCHAR(50)) AS manager_data,
        CAST(NULL AS VARCHAR(50)) AS engineer_info,
        'employee' AS type
    FROM employee
    UNION ALL
    SELECT
        manager.id AS id,
        manager.name AS name,
        manager.manager_data AS manager_data,
        CAST(NULL AS VARCHAR(50)) AS engineer_info,
        'manager' AS type
    FROM manager
    UNION ALL
    SELECT
        engineer.id AS id,
        engineer.name AS name,
        CAST(NULL AS VARCHAR(50)) AS manager_data,
        engineer.engineer_info AS engineer_info,
        'engineer' AS type
    FROM engineer
) AS pjoin
```

Приведенный выше запрос **UNION** должен создавать столбцы `«NULL»` для каждой подтаблицы, чтобы разместить те столбцы, которые не являются членами этого конкретного подкласса.

### Абстрактные конкретные классы

Конкретные сопоставления, проиллюстрированные до сих пор, показывают как подклассы, так и базовый класс, сопоставленные с отдельными таблицами. В конкретном случае использования наследования обычно базовый класс не представлен в базе данных, а только подклассы. Другими словами, базовый класс является **«абстрактным»**.

Обычно, когда нужно сопоставить два разных подкласса с отдельными таблицами и оставить базовый класс неотображенным, это может быть достигнуто очень легко. При использовании Declarative просто объявите базовый класс с индикатором **\_\_abstract\_\_**:

```python
class Employee(Base):
    __abstract__ = True

class Manager(Employee):
    __tablename__ = 'manager'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    manager_data = Column(String(40))

    __mapper_args__ = {
        'polymorphic_identity': 'manager',
    }

class Engineer(Employee):
    __tablename__ = 'engineer'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    engineer_info = Column(String(40))

    __mapper_args__ = {
        'polymorphic_identity': 'engineer',
    }
```

Выше мы на самом деле не используем средства сопоставления наследования SQLAlchemy; мы можем нормально загружать и сохранять экземпляры **Manager** и **Engineer**. Однако ситуация меняется, когда нам нужно выполнить **полиморфный запрос**, то есть мы хотели бы сгенерировать `session.query(Employee)` и получить набор экземпляров **Manager** и **Engineer**. Это возвращает нас в область конкретного наследования, и мы должны создать специальный преобразователь для **Employee**, чтобы добиться этого.

{% hint style="info" %}
**Mappers всегда могут SELECT**

В SQLAlchemy mapper для класса всегда должен ссылаться на некоторый **«selectable»**, который обычно является таблицей [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), но также может ссылаться на любой объект [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select). Хотя может показаться, что преобразователь с «наследованием одиночной таблицы» не сопоставляется с таблицей, на самом деле эти преобразователи неявно ссылаются на таблицу, отображаемую суперклассом.
{% endhint %}

Чтобы изменить наш пример конкретного наследования, чтобы проиллюстрировать «абстрактную» базу, способную к полиморфной загрузке, у нас будет только таблицы **engineer** и **manager** и не будет таблицы **employee**, однако mapper **Employee** будет сопоставлен непосредственно с «полиморфным объединением» вместо того, чтобы указывать его локально в параметре [mapper.with\_polymorphic](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.with\_polymorphic).

Чтобы помочь в этом, Declarative предлагает вариант класса [ConcreteBase](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.ConcreteBase) под названием [AbstractConcreteBase](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.AbstractConcreteBase), который достигает этого автоматически:

```python
from sqlalchemy.ext.declarative import AbstractConcreteBase

class Employee(AbstractConcreteBase, Base):
    pass

class Manager(Employee):
    __tablename__ = 'manager'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    manager_data = Column(String(40))

    __mapper_args__ = {
        'polymorphic_identity': 'manager',
        'concrete': True
    }

class Engineer(Employee):
    __tablename__ = 'engineer'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    engineer_info = Column(String(40))

    __mapper_args__ = {
        'polymorphic_identity': 'engineer',
        'concrete': True
    }
```

Вспомогательный класс [AbstractConcreteBase](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.AbstractConcreteBase) имеет более сложный внутренний процесс, чем у [ConcreteBase](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.ConcreteBase), поскольку полное сопоставление базового класса должно быть отложено до тех пор, пока не будут объявлены все подклассы. При сопоставлении, подобном приведенному выше, могут сохраняться только экземпляры **Manager** и **Engineer**; запросы к классу **Employee** всегда будут создавать объекты **Manager** и **Engineer**.

### Классическая и полуклассическая конкретная полиморфная конфигурация

Декларативные конфигурации, показанные с помощью [ConcreteBase](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.ConcreteBase) и [AbstractConcreteBase](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html#sqlalchemy.ext.declarative.AbstractConcreteBase), эквивалентны двум другим формам конфигурации, которые явно используют [polymorphic\_union()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.polymorphic\_union). Эти конфигурационные формы явно используют объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), поэтому сначала можно создать «полиморфное объединение», а затем применить его к сопоставлениям. Они проиллюстрированы здесь, чтобы прояснить роль функции [polymorphic\_union()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.polymorphic\_union) с точки зрения отображения.

**Полуклассическое отображение**, например, использует декларативное, но устанавливает объекты таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) отдельно:

```python
metadata_obj = Base.metadata

employees_table = Table(
    'employee', metadata_obj,
    Column('id', Integer, primary_key=True),
    Column('name', String(50)),
)

managers_table = Table(
    'manager', metadata_obj,
    Column('id', Integer, primary_key=True),
    Column('name', String(50)),
    Column('manager_data', String(50)),
)

engineers_table = Table(
    'engineer', metadata_obj,
    Column('id', Integer, primary_key=True),
    Column('name', String(50)),
    Column('engineer_info', String(50)),
)
```

Затем **UNION** создается с помощью [polymorphic\_union()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.polymorphic\_union):

```python
from sqlalchemy.orm import polymorphic_union

pjoin = polymorphic_union({
    'employee': employees_table,
    'manager': managers_table,
    'engineer': engineers_table
}, 'type', 'pjoin')
```

С приведенными выше объектами [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) сопоставления могут быть созданы с использованием «полуклассического» стиля, где мы используем Declarative в сочетании с аргументом **\_\_table\_\_**; наш полиморфный союз выше передается через **\_\_mapper\_args\_\_** в параметр [mapper.with\_polymorphic](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.with\_polymorphic):

```python
class Employee(Base):
    __table__ = employee_table
    __mapper_args__ = {
        'polymorphic_on': pjoin.c.type,
        'with_polymorphic': ('*', pjoin),
        'polymorphic_identity': 'employee'
    }

class Engineer(Employee):
    __table__ = engineer_table
    __mapper_args__ = {
        'polymorphic_identity': 'engineer',
        'concrete': True}

class Manager(Employee):
    __table__ = manager_table
    __mapper_args__ = {
        'polymorphic_identity': 'manager',
        'concrete': True}
```

В качестве альтернативы те же самые объекты [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) можно использовать в полностью «классическом» стиле, вообще не используя Declarative. Проиллюстрирован конструктор, аналогичный предоставленному Declarative:

```python
class Employee(object):
    def __init__(self, **kw):
        for k in kw:
            setattr(self, k, kw[k])

class Manager(Employee):
    pass

class Engineer(Employee):
    pass

employee_mapper = mapper_registry.map_imperatively(
    Employee,
    pjoin,
    with_polymorphic=('*', pjoin),
    polymorphic_on=pjoin.c.type,
)
manager_mapper = mapper_registry.map_imperatively(
    Manager,
    managers_table,
    inherits=employee_mapper,
    concrete=True,
    polymorphic_identity='manager',
)
engineer_mapper = mapper_registry.map_imperatively(
    Engineer,
    engineers_table,
    inherits=employee_mapper,
    concrete=True,
    polymorphic_identity='engineer',
)
```

«Абстрактный» пример также может быть отображен в «полуклассическом» или «классическом» стиле. Разница в том, что вместо того, чтобы применять «полиморфное объединение» к параметру [mapper.with\_polymorphic](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.with\_polymorphic), мы применяем его непосредственно как сопоставленный выбор в нашем самом базовом преобразователе mapper. Полуклассическое отображение показано ниже:

```python
from sqlalchemy.orm import polymorphic_union

pjoin = polymorphic_union({
    'manager': managers_table,
    'engineer': engineers_table
}, 'type', 'pjoin')

class Employee(Base):
    __table__ = pjoin
    __mapper_args__ = {
        'polymorphic_on': pjoin.c.type,
        'with_polymorphic': '*',
        'polymorphic_identity': 'employee'
    }

class Engineer(Employee):
    __table__ = engineer_table
    __mapper_args__ = {
        'polymorphic_identity': 'engineer',
        'concrete': True}

class Manager(Employee):
    __table__ = manager_table
    __mapper_args__ = {
        'polymorphic_identity': 'manager',
        'concrete': True}
```

Выше мы используем [polymorphic\_union()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.polymorphic\_union) так же, как и раньше, за исключением того, что мы опускаем таблицу **employee**.

{% hint style="info" %}
Смотри также:

[Императивные (также известные как классические) сопоставления](../otobrazhenie-klassov-python.md#imperativnoe-ono-zhe-klassicheskoe-sopostavlenie-imperative-mapping) - справочная информация о «классических» сопоставлениях.
{% endhint %}

### Отношения с конкретным наследием

В конкретном сценарии наследования сопоставление отношений является сложной задачей, поскольку отдельные классы не используют общую таблицу. Если отношения включают только определенные классы, такие как отношение между **Company** в наших предыдущих примерах и **Manager**, специальные шаги не требуются, поскольку это всего лишь две связанные таблицы.

Однако, если **Company** должна иметь отношение «один ко многим» к **Employee**, указывающее, что коллекция может включать объекты **Engineer** и **Manager**, это означает, что **Employee** должен иметь возможности полиморфной загрузки, а также что каждая связанная таблица должна иметь внешний ключ обратный к таблице **Company**. Пример такой конфигурации выглядит следующим образом:

```python
from sqlalchemy.ext.declarative import ConcreteBase


class Company(Base):
    __tablename__ = 'company'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    employees = relationship("Employee")


class Employee(ConcreteBase, Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    company_id = Column(ForeignKey('company.id'))

    __mapper_args__ = {
        'polymorphic_identity': 'employee',
        'concrete': True
    }


class Manager(Employee):
    __tablename__ = 'manager'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    manager_data = Column(String(40))
    company_id = Column(ForeignKey('company.id'))

    __mapper_args__ = {
        'polymorphic_identity': 'manager',
        'concrete': True
    }


class Engineer(Employee):
    __tablename__ = 'engineer'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    engineer_info = Column(String(40))
    company_id = Column(ForeignKey('company.id'))

    __mapper_args__ = {
        'polymorphic_identity': 'engineer',
        'concrete': True
    }
```

Следующая сложность с конкретным наследованием и отношениями возникает, когда мы хотим, чтобы один или все **Employee**, **Manager** и **Engineer** сами ссылались на **Company**. В этом случае SQLAlchemy имеет особое поведение, заключающееся в том, что [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), помещенное в **Employee**, которое ссылается на **Company**, не работает с классами **Manager** и **Engineer** при выполнении на уровне экземпляра. Вместо этого к каждому классу должно применяться отдельное [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship). Чтобы добиться двунаправленного поведения с точки зрения трех отдельных отношений, которые служат противоположностью **Company.employees**, между каждым из отношений используется [relationship.back\_populates](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates):

```python
from sqlalchemy.ext.declarative import ConcreteBase


class Company(Base):
    __tablename__ = 'company'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    employees = relationship("Employee", back_populates="company")


class Employee(ConcreteBase, Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    company_id = Column(ForeignKey('company.id'))
    company = relationship("Company", back_populates="employees")

    __mapper_args__ = {
        'polymorphic_identity': 'employee',
        'concrete': True
    }


class Manager(Employee):
    __tablename__ = 'manager'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    manager_data = Column(String(40))
    company_id = Column(ForeignKey('company.id'))
    company = relationship("Company", back_populates="employees")

    __mapper_args__ = {
        'polymorphic_identity': 'manager',
        'concrete': True
    }


class Engineer(Employee):
    __tablename__ = 'engineer'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    engineer_info = Column(String(40))
    company_id = Column(ForeignKey('company.id'))
    company = relationship("Company", back_populates="employees")

    __mapper_args__ = {
        'polymorphic_identity': 'engineer',
        'concrete': True
    }
```

Вышеупомянутое ограничение связано с текущей реализацией, в том числе с тем, что конкретные классы-наследники не имеют общих атрибутов суперкласса и, следовательно, требуют установки отдельных отношений.

### Загрузка конкретных сопоставлений наследования

Варианты загрузки с конкретным наследованием ограничены; как правило, если полиморфная загрузка настроена в преобразователе с использованием одного из декларативных конкретных миксинов, ее нельзя изменить во время запроса в текущих версиях SQLAlchemy. Обычно функция [with\_polymorphic()](https://docs.sqlalchemy.org/en/14/orm/inheritance\_loading.html#sqlalchemy.orm.with\_polymorphic) может переопределить стиль загрузки, используемый конкретным сопоставлением, однако из-за текущих ограничений это пока не поддерживается.
