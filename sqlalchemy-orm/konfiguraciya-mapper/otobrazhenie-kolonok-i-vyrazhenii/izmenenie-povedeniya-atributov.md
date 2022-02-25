# Изменение поведения атрибутов

## Простые валидаторы

Быстрый способ добавить процедуру «валидации» к атрибуту — использовать декоратор [validates()](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.validates). Валидатор атрибута может вызвать исключение, остановив процесс изменения значения атрибута, или может изменить данное значение на что-то другое. **Валидаторы**, как и все расширения атрибутов, **вызываются только обычным пользовательским кодом; они не выдаются, когда ORM заполняет объект**:

```python
from sqlalchemy.orm import validates

class EmailAddress(Base):
    __tablename__ = 'address'

    id = Column(Integer, primary_key=True)
    email = Column(String)

    @validates('email')
    def validate_email(self, key, address):
        if '@' not in address:
            raise ValueError("failed simple email validation")
        return address
```

{% hint style="warning" %}
**Изменено в версии 1.0.0**: — валидаторы больше не запускаются в процессе сброса flush, когда извлекаются вновь полученные значения для столбцов первичного ключа, а также некоторые значения по умолчанию на стороне Python или на стороне сервера. До версии 1.0 в этих случаях также могли срабатывать валидаторы.
{% endhint %}

Валидаторы также получают события добавления коллекции, когда элементы добавляются в коллекцию:

```python
from sqlalchemy.orm import validates

class User(Base):
    # ...

    addresses = relationship("Address")

    @validates('addresses')
    def validate_address(self, key, address):
        if '@' not in address.email:
            raise ValueError("failed simplified email validation")
        return address
```

Функция проверки по умолчанию не генерируется для событий удаления коллекции, поскольку обычно ожидается, что отбрасываемое значение не требует проверки. Тем не менее, [validates()](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.validates) поддерживает прием этих событий, указав `include_removes=True` в декораторе. Когда этот флаг установлен, функция проверки должна получить дополнительный логический аргумент, который, если он равен `True`, указывает, что операция является удалением:

```python
from sqlalchemy.orm import validates

class User(Base):
    # ...

    addresses = relationship("Address")

    @validates('addresses', include_removes=True)
    def validate_address(self, key, address, is_remove):
        if is_remove:
            raise ValueError(
                    "not allowed to remove items from the collection")
        else:
            if '@' not in address.email:
                raise ValueError("failed simplified email validation")
            return address
```

Случай, когда взаимозависимые валидаторы связаны через обратную ссылку, также можно настроить, используя опцию `include_backrefs=False`; этот параметр, если установлено значение `False`, предотвращает запуск функции проверки, если событие происходит в результате обратной ссылки **backref**:

```python
from sqlalchemy.orm import validates

class User(Base):
    # ...

    addresses = relationship("Address", backref='user')

    @validates('addresses', include_backrefs=False)
    def validate_address(self, key, address):
        if '@' not in address:
            raise ValueError("failed simplified email validation")
        return address
```

Выше, если бы мы присвоили `Address.user`, как в `some_address.user = some_user`, функция `validate_address()` не сгенерировалась бы, даже если происходит добавление к `some_user.addresses` — событие вызвано обратной ссылкой.

Обратите внимание, что декоратор [validates()](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.validates) — это удобная функция, построенная поверх событий атрибутов. Приложение, которое требует большего контроля над конфигурацией поведения изменения атрибута, может использовать эту систему, описанную в [AttributeEvents](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.AttributeEvents).

| Имя объекта                                                                                                          | Описание                                                                        |
| -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| [validates](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.validates)(\*names, \*\*kw) | Декорирует метод как «валидатор» для одного или нескольких именованных свойств. |

### _function_ sqlalchemy.orm.validates(_\*names_, _\*\*kw_)

Декорирует метод как «валидатор» для одного или нескольких именованных свойств.

Назначает метод в качестве валидатора, метод, который получает имя атрибута, а также присваиваемое значение или, в случае коллекции, значение, которое нужно добавить в коллекцию. Затем функция может вызвать исключение проверки, чтобы остановить процесс (где встроенные в Python исключения **ValueError** и **AssertionError** являются разумным выбором), или может изменить или заменить значение перед продолжением. В противном случае функция должна возвращать заданное значение.

Обратите внимание, что валидатор для коллекции **не может** загружать эту коллекцию в рамках процедуры проверки — это использование вызывает утверждение, чтобы избежать переполнения рекурсии. Это условие повторного входа, которое не поддерживается.

#### Параметры

* **\*names** - список имен атрибутов, которые необходимо проверить.
* **include\_removes** - если `True`, также будут отправлены события «удалить» - функция проверки должна принимать дополнительный аргумент `«is_remove»`, который будет логическим.
* **include\_backrefs** - по умолчанию `True`; если `False`, функция проверки не будет выдаваться, если инициатором является событие атрибута, связанное через обратную ссылку. Это можно использовать для двунаправленного использования [validates()](izmenenie-povedeniya-atributov.md#function-sqlalchemy.orm.validates-names-kw), когда только один валидатор должен генерировать для каждой операции с атрибутом.

{% hint style="warning" %}
**Новое в версии 0.9.0.**
{% endhint %}

{% hint style="info" %}
Смотри также:

[Простые валидаторы](izmenenie-povedeniya-atributov.md#prostye-validatory) — примеры использования [validates()](izmenenie-povedeniya-atributov.md#function-sqlalchemy.orm.validates-names-kw)
{% endhint %}

## Использование пользовательских типов данных на базовом уровне Core

Не-ORM средства воздействия на значение столбца способом, подходящим для преобразования данных между тем, как они представлены в Python, и тем, как они представлены в базе данных, могут быть достигнуты с помощью пользовательского типа данных, который применяется к сопоставленные метаданные таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table). Это более распространено в случае некоторого стиля кодирования/декодирования, которое происходит как при отправке данных в базу данных, так и при их возврате; подробнее об этом читайте в документации Core в дополнении к существующим типам ([Augmenting Existing Types](https://docs.sqlalchemy.org/en/14/core/custom\_types.html#types-typedecorator)).

## Использование дескрипторов и гибридов

Более комплексный способ изменить поведение атрибута — использовать [дескрипторы](https://docs.sqlalchemy.org/en/14/glossary.html#term-descriptors). Они обычно используются в Python с помощью функции `property()`. Стандартный метод SQLAlchemy для дескрипторов заключается в создании простого дескриптора и его чтении/записи из сопоставленного атрибута с другим именем. Ниже мы проиллюстрируем это с помощью свойств в стиле Python 2.6:

```python
class EmailAddress(Base):
    __tablename__ = 'email_address'

    id = Column(Integer, primary_key=True)

    # назовите атрибут символом подчеркивания,
    # отличным от имени столбца
    _email = Column("email", String)

    # затем создайте атрибут «.email»,
    # чтобы получить/установить «._email»
    @property
    def email(self):
        return self._email

    @email.setter
    def email(self, email):
        self._email = email
```

Описанный выше подход будет работать, но мы можем добавить еще кое-что. В то время как наш объект **EmailAddress** будет перемещать значение через дескриптор **email** в сопоставленный атрибут **\_email**, атрибут `EmailAddress.email` на уровне класса не имеет обычной семантики выражений, используемой с [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query). Чтобы предоставить их, мы вместо этого используем [гибридное](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html#module-sqlalchemy.ext.hybrid) расширение следующим образом:

```python
from sqlalchemy.ext.hybrid import hybrid_property

class EmailAddress(Base):
    __tablename__ = 'email_address'

    id = Column(Integer, primary_key=True)

    _email = Column("email", String)

    @hybrid_property
    def email(self):
        return self._email

    @email.setter
    def email(self, email):
        self._email = email
```

Атрибут `.email`, в дополнение к обеспечению поведения получателя/установщика, когда у нас есть экземпляр **EmailAddress**, также предоставляет выражение SQL при использовании на уровне класса, то есть непосредственно из класса **EmailAddress**:

```python
from sqlalchemy.orm import Session
session = Session()

address = session.query(EmailAddress).\
                 filter(EmailAddress.email == 'address@example.com').\
                 one()
```

```sql
SELECT address.email AS address_email, address.id AS address_id
FROM address
WHERE address.email = ?
('address@example.com',)
```

```python
address.email = 'otheraddress@example.com'
session.commit()
```

```sql
UPDATE address SET email=? WHERE address.id = ?
('otheraddress@example.com', 1)
COMMIT
```

[`hybrid_property`](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html#sqlalchemy.ext.hybrid.hybrid\_property) также позволяет нам изменить поведение атрибута, в том числе определить отдельное поведение при доступе к атрибуту на уровне экземпляра, а не на уровне класса/выражения, с помощью модификатора [hybrid\_property.expression()](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html#sqlalchemy.ext.hybrid.hybrid\_property.expression). Например, если бы мы хотели добавить имя хоста автоматически, мы могли бы определить два набора логики манипуляции со строками:

```python
class EmailAddress(Base):
    __tablename__ = 'email_address'

    id = Column(Integer, primary_key=True)

    _email = Column("email", String)

    @hybrid_property
    def email(self):
        """Возвращает значение _email до последних 12 символов."""

        return self._email[:-12]

    @email.setter
    def email(self, email):
        """Установит значение _email, добавив 12-значное значение @example.com."""

        self._email = email + "@example.com"

    @email.expression
    def email(cls):
        """Создайте выражение SQL, представляющее значение столбца _email
        за вычетом последних двенадцати символов."""

        return func.substr(cls._email, 0, func.length(cls._email) - 12)
```

Выше, доступ к свойству **email** экземпляра **EmailAddress** вернет значение атрибута **\_email**, удалив или добавив имя хоста `@example.com` из значения. Когда мы запрашиваем атрибут **email**, отображается функция SQL, которая производит тот же эффект:

```python
address = session.query(EmailAddress).filter(EmailAddress.email == 'address').one()
```

```sql
SELECT address.email AS address_email, address.id AS address_id
FROM address
WHERE substr(address.email, ?, length(address.email) - ?) = ?
(0, 12, 'address')
```

Узнайте больше о гибридах в [Hybrid Attributes](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html).

## Синонимы

Синонимы — это конструкция уровня преобразователя mapper, которая позволяет любому атрибуту класса «отражать» другой отображаемый атрибут.

В самом общем смысле синоним — это простой способ сделать определенный атрибут доступным по дополнительному имени:

```python
from sqlalchemy.orm import synonym

class MyClass(Base):
    __tablename__ = 'my_table'

    id = Column(Integer, primary_key=True)
    job_status = Column(String(50))

    status = synonym("job_status")
```

Приведенный выше класс **MyClass** имеет два атрибута, `.job_status` и `.status`, которые будут вести себя как один атрибут, оба на уровне выражения:

```python
>>> print(MyClass.job_status == 'some_status')
my_table.job_status = :job_status_1

>>> print(MyClass.status == 'some_status')
my_table.job_status = :job_status_1
```

и на уровне экземпляра:

```python
>>> m1 = MyClass(status='x')
>>> m1.status, m1.job_status
('x', 'x')

>>> m1.job_status = 'y'
>>> m1.status, m1.job_status
('y', 'y')
```

[synonym()](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.synonym) можно использовать для любого типа сопоставленного атрибута, который является подклассом [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty), включая сопоставленные столбцы и отношения, а также сами синонимы.

Помимо простого зеркала, [synonym()](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.synonym) также может ссылаться на определяемый пользователем [дескриптор](https://docs.sqlalchemy.org/en/14/glossary.html#term-descriptor). Мы можем снабдить наш синоним статуса `@property`:

```python
class MyClass(Base):
    __tablename__ = 'my_table'

    id = Column(Integer, primary_key=True)
    status = Column(String(50))

    @property
    def job_status(self):
        return "Status: " + self.status

    job_status = synonym("status", descriptor=job_status)
```

При использовании Declarative приведенный выше шаблон можно выразить более кратко, используя декоратор [synonym\_for()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.synonym\_for):

```python
from sqlalchemy.ext.declarative import synonym_for

class MyClass(Base):
    __tablename__ = 'my_table'

    id = Column(Integer, primary_key=True)
    status = Column(String(50))

    @synonym_for("status")
    @property
    def job_status(self):
        return "Status: " + self.status
```

В то время как [synonym()](izmenenie-povedeniya-atributov.md#function-sqlalchemy.orm.synonym-name-map\_column-none-descriptor-none-comparator\_factory-none-doc-non) полезен для простого зеркального отображения, вариант использования расширения поведения атрибута с помощью дескрипторов лучше обрабатывается в современном использовании с использованием функции [гибридных атрибутов](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#mapper-hybrids), которая больше ориентирована на дескрипторы Python. Технически [synonym()](izmenenie-povedeniya-atributov.md#function-sqlalchemy.orm.synonym-name-map\_column-none-descriptor-none-comparator\_factory-none-doc-non) может делать все то же, что и [hybrid\_property](https://docs.sqlalchemy.org/en/14/orm/extensions/hybrid.html#sqlalchemy.ext.hybrid.hybrid\_property), так как он также поддерживает внедрение пользовательских возможностей SQL, но гибрид проще использовать в более сложных ситуациях.

| Имя объекта                                                                                                                                                 | Описание                                                                                                                                        |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| [synonym](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.synonym)(name\[, map\_column, descriptor, comparator\_factory, ...]) | Обозначьте имя атрибута как синоним сопоставленного свойства, поскольку атрибут будет отражать поведение значения и выражения другого атрибута. |

### _function_ sqlalchemy.orm.synonym(_name_, _map\_column=None_, _descriptor=None_, _comparator\_factory=None_, _doc=None_, _info=None_)

Обозначьте имя атрибута как синоним сопоставленного свойства, поскольку атрибут будет отражать поведение значения и выражения другого атрибута. Например.:

```python
class MyClass(Base):
    __tablename__ = 'my_table'

    id = Column(Integer, primary_key=True)
    job_status = Column(String(50))

    status = synonym("job_status")
```

#### Параметры:

* **name** - имя существующего сопоставленного свойства. Это может относиться к строковому имени атрибута ORM-сопоставления, настроенному для класса, включая атрибуты и отношения, привязанные к столбцу.
* **descriptor** - [дескриптор](https://docs.sqlalchemy.org/en/14/glossary.html#term-descriptor) Python, который будет использоваться как getter (и, возможно, setter) при доступе к этому атрибуту на уровне экземпляра.
* **map\_column** - **Только для классических сопоставлений и сопоставлений с существующим объектом Table.** Если `True`, конструкция [synonym()](izmenenie-povedeniya-atributov.md#function-sqlalchemy.orm.synonym-name-map\_column-none-descriptor-none-comparator\_factory-none-doc-non) найдет объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) в сопоставленной таблице, которая обычно связана с именем атрибута этого синонима, и создаст новое свойство [ColumnProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.ColumnProperty), которое вместо этого сопоставляет этот столбец [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) с альтернативным именем, заданным как `«name»` , аргумент синонима; таким образом, обычный шаг переопределения сопоставления столбца [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) под другим именем становится излишним. Обычно это предназначено для использования, когда столбец должен быть заменен атрибутом, который также использует дескриптор, то есть в сочетании с параметром [synonym.descriptor](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#sqlalchemy.orm.synonym.params.descriptor):

```python
my_table = Table(
    "my_table", metadata,
    Column('id', Integer, primary_key=True),
    Column('job_status', String(50))
)

class MyClass(object):
    @property
    def _job_status_descriptor(self):
        return "Status: %s" % self._job_status


mapper(
    MyClass, my_table, properties={
        "job_status": synonym(
            "_job_status", map_column=True,
            descriptor=MyClass._job_status_descriptor)
    }
)
```

Выше атрибут с именем **\_job\_status** автоматически сопоставляется со столбцом **job\_status**:

```python
>>> j1 = MyClass()
>>> j1._job_status = "employed"
>>> j1.job_status
Status: employed
```

При использовании Declarative для предоставления дескриптора в сочетании с синонимом используйте вспомогательную функцию `sqlalchemy.ext.declarative.synonym_for()`. Однако обратите внимание, что функция [гибридных свойств](https://docs.sqlalchemy.org/en/14/orm/mapped\_attributes.html#mapper-hybrids) обычно предпочтительнее, особенно при переопределении поведения атрибута.

* **info** - Необязательный словарь данных, который будет заполнен атрибутом `InspectionAttr.info` этого объекта.

{% hint style="info" %}
**Новое в версии 1.0.0.**
{% endhint %}

* **comparator\_factory** - Подкласс [PropComparator](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator), обеспечивающий настраиваемое поведение сравнения на уровне выражения SQL.

{% hint style="info" %}
В случае использования предоставления атрибута, который переопределяет поведение атрибута как на уровне Python, так и на уровне SQL-выражения, обратитесь к атрибуту Hybrid, представленному в разделе [Использование дескрипторов и гибридов](izmenenie-povedeniya-atributov.md#ispolzovanie-deskriptorov-i-gibridov) для более эффективного метода.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Синонимы](izmenenie-povedeniya-atributov.md#sinonimy) - Обзор синонимов

[synonym\_for()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.synonym\_for) - помощник, ориентированный на декларативный

[Использование дескрипторов и гибридов](izmenenie-povedeniya-atributov.md#ispolzovanie-deskriptorov-i-gibridov). Расширение Hybrid Attribute обеспечивает обновленный подход к расширению поведения атрибутов более гибко, чем это может быть достигнуто с помощью синонимов.
{% endhint %}

## Индивидуальная настройка оператора

«Операторы», используемые SQLAlchemy ORM и языком выражений Core, полностью настраиваются. Например, в выражении сравнения `User.name == 'ed'` используется оператор, встроенный в сам Python, который называется `operator.eq` — фактическая конструкция SQL, которую SQLAlchemy связывает с таким оператором, может быть изменена. Новые операции также могут быть связаны с выражениями столбцов. Операторы, используемые для выражений столбцов, наиболее непосредственно переопределяются на уровне типа — описание см. в разделе «Переопределение и создание новых операторов» ([Redefining and Creating New Operators](https://docs.sqlalchemy.org/en/14/core/custom\_types.html#types-operators)).

Функции уровня ORM, такие как [column\_property()](otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs), [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) и [composite()](https://docs.sqlalchemy.org/en/14/orm/composites.html#sqlalchemy.orm.composite), также обеспечивают переопределение оператора на уровне ORM, передавая подкласс [PropComparator](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator) в аргумент `comparator_factory` каждой функции. Настройка операторов на этом уровне является редким вариантом использования. См. документацию на [PropComparator](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator) для обзора.
