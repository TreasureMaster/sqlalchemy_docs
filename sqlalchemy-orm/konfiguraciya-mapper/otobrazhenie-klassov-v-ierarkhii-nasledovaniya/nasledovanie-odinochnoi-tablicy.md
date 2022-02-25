# Наследование одиночной таблицы

## Наследование одиночной таблицы

Наследование одиночной таблицы представляет все атрибуты всех подклассов в одной таблице. Конкретный подкласс с атрибутами, уникальными для этого класса, сохранит их в столбцах таблицы, которые в противном случае равны **NULL**, если строка относится к другому типу объекта.

Запрос определенного подкласса в иерархии будет отображаться как **SELECT** по отношению к базовой таблице, которая будет включать предложение **WHERE**, которое ограничивает строки теми, которые имеют определенное значение или значения, присутствующие в столбце или выражении дискриминатора.

Преимущество наследования одиночной таблицы заключается в простоте по сравнению с наследованием объединенных таблиц; запросы намного эффективнее, поскольку для загрузки объектов каждого представленного класса необходимо задействовать только одну таблицу.

Конфигурация наследования одиночной таблицы очень похожа на наследование объединенных таблиц, за исключением того, что базовый класс указывает **\_\_tablename\_\_**. В базовой таблице также требуется столбец дискриминатора, чтобы классы можно было отличить друг от друга.

Несмотря на то, что подклассы совместно используют базовую таблицу для всех своих атрибутов, при использовании декларативных объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) все еще могут быть указаны в подклассах, указывая, что столбец должен быть сопоставлен только с этим подклассом; [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) будет применен к тому же базовому объекту [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table):

```python
class Employee(Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    type = Column(String(20))

    __mapper_args__ = {
        'polymorphic_on':type,
        'polymorphic_identity':'employee'
    }

class Manager(Employee):
    manager_data = Column(String(50))

    __mapper_args__ = {
        'polymorphic_identity':'manager'
    }

class Engineer(Employee):
    engineer_info = Column(String(50))

    __mapper_args__ = {
        'polymorphic_identity':'engineer'
    }
```

Обратите внимание, что преобразователи для производных классов **Manager** и **Engineer** опускают **\_\_tablename\_\_**, указывая на то, что у них нет собственной отображаемой таблицы.

### Разрешение конфликтов столбцов

Обратите внимание в предыдущем разделе, что столбцы **manager\_name** и **engineering\_info** «перемещены вверх» для применения к **Employee.\_\_table\_\_** в результате их объявления в подклассе, у которого нет собственной таблицы. Возникает сложный случай, когда два подкласса хотят указать _один и тот же_ столбец, как показано ниже:

```python
class Employee(Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    type = Column(String(20))

    __mapper_args__ = {
        'polymorphic_on':type,
        'polymorphic_identity':'employee'
    }

class Engineer(Employee):
    __mapper_args__ = {'polymorphic_identity': 'engineer'}
    start_date = Column(DateTime)

class Manager(Employee):
    __mapper_args__ = {'polymorphic_identity': 'manager'}
    start_date = Column(DateTime)
```

Выше столбец **start\_date**, объявленный как для **Engineer**, так и для **Manager**, приведет к ошибке:

```bash
sqlalchemy.exc.ArgumentError: Column 'start_date' on class
<class '__main__.Manager'> conflicts with existing
column 'employee.start_date'
```

Приведенный выше сценарий представляет собой двусмысленность в системе декларативного сопоставления, которая может быть разрешена с помощью [declared\_attr](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.declared\_attr) для условного определения столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), позаботившись о возврате существующего столбца через родительскую **\_\_table\_\_**, если он уже существует:

```python
from sqlalchemy.orm import declared_attr

class Employee(Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    type = Column(String(20))

    __mapper_args__ = {
        'polymorphic_on':type,
        'polymorphic_identity':'employee'
    }

class Engineer(Employee):
    __mapper_args__ = {'polymorphic_identity': 'engineer'}

    @declared_attr
    def start_date(cls):
        "Start date column, if not present already."
        return Employee.__table__.c.get('start_date', Column(DateTime))

class Manager(Employee):
    __mapper_args__ = {'polymorphic_identity': 'manager'}

    @declared_attr
    def start_date(cls):
        "Start date column, if not present already."
        return Employee.__table__.c.get('start_date', Column(DateTime))
```

Выше при отображении **Manager** столбец **start\_date** уже присутствует в классе **Employee**; возвращая существующий объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), декларативная система распознает, что это один и тот же столбец, который должен быть сопоставлен с двумя разными подклассами по отдельности.

Аналогичную концепцию можно использовать с классами примесей (см. [Составление сопоставленных иерархий с помощью примесей](../otobrazhenie-klassov-v-deklarativnom-stile/sostavlenie-sopostavlennykh-ierarkhii-s-pomoshyu-miksinov.md)) для определения определенной серии столбцов и/или других сопоставленных атрибутов из повторно используемого класса примесей:

```python
class Employee(Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    type = Column(String(20))

    __mapper_args__ = {
        'polymorphic_on':type,
        'polymorphic_identity':'employee'
    }

class HasStartDate:
    @declared_attr
    def start_date(cls):
        return cls.__table__.c.get('start_date', Column(DateTime))

class Engineer(HasStartDate, Employee):
    __mapper_args__ = {'polymorphic_identity': 'engineer'}

class Manager(HasStartDate, Employee):
    __mapper_args__ = {'polymorphic_identity': 'manager'}
```

### Отношения с наследованием одиночной таблицы

Отношения полностью поддерживаются наследованием одной таблицы. Конфигурация выполняется так же, как и при объединенном наследовании; атрибут внешнего ключа должен находиться в том же классе, что и «внешняя» сторона отношения:

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
    manager_data = Column(String(50))

    __mapper_args__ = {
        'polymorphic_identity':'manager'
    }

class Engineer(Employee):
    engineer_info = Column(String(50))

    __mapper_args__ = {
        'polymorphic_identity':'engineer'
    }
```

Кроме того, как и в случае объединенного наследования, мы можем создавать отношения, включающие определенный подкласс. При запросе оператор **SELECT** будет включать предложение **WHERE**, которое ограничивает выбор класса этим подклассом или подклассами:

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
    manager_name = Column(String(30))

    company_id = Column(ForeignKey('company.id'))
    company = relationship("Company", back_populates="managers")

    __mapper_args__ = {
        'polymorphic_identity':'manager',
    }


class Engineer(Employee):
    engineer_info = Column(String(50))

    __mapper_args__ = {
        'polymorphic_identity':'engineer'
    }
```

Выше класс **Manager** будет иметь атрибут **Manager.company**; **Company** будет иметь атрибут **Company.managers**, который всегда загружается для сотрудника с дополнительным предложением **WHERE**, которое ограничивает строки строками с `type = 'manager'`.

### Загрузка одиночных сопоставлений наследования

Методы загрузки для наследования одиночной таблицы в основном идентичны тем, которые используются для наследования объединенных таблиц, и между этими двумя типами отображения обеспечивается высокая степень абстракции, так что между ними легко переключаться, а также смешивать их в единую иерархию (просто опустите **\_\_tablename\_\_** в зависимости от того, какие подклассы должны наследоваться по одному). См. разделы Загрузка иерархий наследования ([Loading Inheritance Hierarchies](https://docs.sqlalchemy.org/en/14/orm/inheritance\_loading.html)) и Загрузка объектов с наследованием одиночной таблицы ([Loading objects with single table inheritance](https://docs.sqlalchemy.org/en/14/orm/inheritance\_loading.html#loading-single-inheritance)) для получения документации по методам загрузки наследования, включая настройку классов, которые необходимо запрашивать как во время настройки преобразователя mapper, так и во время запроса.
