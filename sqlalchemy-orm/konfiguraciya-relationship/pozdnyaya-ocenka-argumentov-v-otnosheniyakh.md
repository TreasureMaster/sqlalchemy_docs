# Поздняя оценка аргументов в отношениях

## Поздняя оценка аргументов в отношениях

Многие примеры в предыдущих разделах иллюстрируют сопоставления, в которых различные конструкции отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) ссылаются на свои целевые классы, используя строковое имя, а не сам класс:

```python
class Parent(Base):
    # ...

    children = relationship("Child", back_populates="parent")

class Child(Base):
    # ...

    parent = relationship("Parent", back_populates="children")
```

Эти строковые имена преобразуются в классы на этапе разрешения преобразователя, который представляет собой внутренний процесс, который обычно происходит после определения всех сопоставлений и обычно запускается при первом использовании самих сопоставлений. Объект реестра [registry](../konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl) — это контейнер, в котором эти имена хранятся и разрешаются в сопоставленные классы, на которые они ссылаются.

В дополнение к основному аргументу класса для отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), другие аргументы, которые зависят от столбцов, присутствующих в пока еще неопределенном классе, также могут быть указаны либо как функции Python, либо, чаще, как строки. Для большинства этих аргументов, за исключением основного аргумента, входные строки **оцениваются как выражения Python с использованием встроенной в Python функции eval()**, поскольку они предназначены для получения полных выражений SQL.

{% hint style="danger" %}
Поскольку функция Python **eval()** используется для интерпретации строковых аргументов с поздней оценкой, переданных в конфигурационную конструкцию модуля отображения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), эти аргументы **не следует** переназначать таким образом, чтобы они получали ненадежный пользовательский ввод; `eval()` **не защищен** от ненадежного пользовательского ввода.
{% endhint %}

Полное пространство имен, доступное в этой оценке, включает все классы, сопоставленные для этой декларативной базы, а также содержимое пакета **sqlalchemy**, включая функции выражений, такие как [desc()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.desc) и `sqlalchemy.sql.functions.func`:

```python
class Parent(Base):
    # ...

    children = relationship(
        "Child",
        order_by="desc(Child.email_address)",
        primaryjoin="Parent.id == Child.parent_id"
    )
```

В случае, когда более чем один модуль содержит класс с одним и тем же именем, имена строковых классов также могут быть указаны как пути с указанием модуля в любом из этих строковых выражений:

```python
class Parent(Base):
    # ...

    children = relationship(
        "myapp.mymodel.Child",
        order_by="desc(myapp.mymodel.Child.email_address)",
        primaryjoin="myapp.mymodel.Parent.id == myapp.mymodel.Child.parent_id"
    )
```

Квалифицированный путь может быть любым частичным путем, который устраняет двусмысленность между именами. Например, чтобы устранить неоднозначность между **myapp.model1.Child** и **myapp.model2.Child**, мы можем указать **model1.Child** или **model2.Child**:

```python
class Parent(Base):
    # ...

    children = relationship(
        "model1.Child",
        order_by="desc(mymodel1.Child.email_address)",
        primaryjoin="Parent.id == model1.Child.parent_id"
    )
```

Конструкция [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) также принимает функции Python или лямбда-выражения в качестве входных данных для этих аргументов. Это имеет то преимущество, что обеспечивает большую безопасность во время компиляции и лучшую поддержку сценариев IDE и [PEP 484](https://www.python.org/dev/peps/pep-0484).

Функциональный подход Python может выглядеть следующим образом:

```python
from sqlalchemy import desc

def _resolve_child_model():
     from myapplication import Child
     return Child

class Parent(Base):
    # ...

    children = relationship(
        _resolve_child_model(),
        order_by=lambda: desc(_resolve_child_model().email_address),
        primaryjoin=lambda: Parent.id == _resolve_child_model().parent_id
    )
```

Полный список параметров, которые принимают Python функции/лямбды или строки, которые будут переданы в **eval()**:

* [`relationship.order_by`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.order\_by)
* [`relationship.primaryjoin`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin)
* [`relationship.secondaryjoin`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondaryjoin)
* [`relationship.secondary`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary)
* [`relationship.remote_side`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.remote\_side)
* [`relationship.foreign_keys`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.foreign\_keys)
* [`relationship._user_defined_foreign_keys`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.\_user\_defined\_foreign\_keys)

{% hint style="warning" %}
**Изменено в версии 1.3.16**: До SQLAlchemy 1.3.16 основной аргумент [relationship.argument](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.argument) для отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) также оценивался с помощью **`eval()`** Начиная с версии 1.3.16 имя строки разрешается непосредственно из преобразователя класса без поддержки пользовательских выражений Python. .
{% endhint %}

{% hint style="danger" %}
Как указывалось ранее, приведенные выше параметры отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) **оцениваются как выражения кода Python с использованием eval(). НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЕ ВВОДНЫЕ СВЕДЕНИЯ В ЭТИ АРГУМЕНТЫ**.
{% endhint %}

Следует также отметить, что аналогично тому, как описано в разделе [Добавление новых столбцов](../konfiguraciya-mapper/otobrazhenie-klassov-v-deklarativnom-stile/tablichnaya-konfiguraciya-v-deklarativnom-otobrazhenii.md#dobavlenie-novykh-kolonok), любая конструкция [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty) может быть добавлена к декларативному базовому сопоставлению в любое время. Если бы мы хотели реализовать это отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) после того, как класс **Address** стал доступен, мы могли бы также применить его позже:

```python
# во-первых, это модуль A, где Child еще не создан,
# мы создаем здесь класс Parent, который ничего не знает о Child

class Parent(Base):
    # ...


#... позже в модуле B, который импортируется после модуля A:

class Child(Base):
    # ...

from module_a import Parent

# назначаем отношение User.addresses в качестве переменной класса.
# Декларативный базовый класс перехватит это и сопоставит отношение.
Parent.children = relationship(
    Child,
    primaryjoin=Child.parent_id==Parent.id
)
```

{% hint style="info" %}
назначение сопоставленных свойств декларативно сопоставленному классу будет работать правильно только в том случае, если используется «декларативный базовый» класс, который также предоставляет управляемый метаклассом метод **\_\_setattr\_\_()**, который будет перехватывать эти операции. Это **не сработает**, если используется декларативный декоратор, предоставленный [registry.mapped()](../konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md#method-sqlalchemy.orm.registry.mapped-cls), и не будет работать для императивно сопоставленного класса, сопоставленного с помощью [registry.map\_imperatively()](../konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md#method-sqlalchemy.orm.registry.map\_imperatively-class\_-local\_table-none-kw).
{% endhint %}

## Поздняя оценка отношения «многие ко многим»

Отношения «многие ко многим» включают ссылку на дополнительный, обычно не отображаемый объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), который обычно присутствует в коллекции метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), на которую ссылается реестр [registry](../konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md). Система поздней оценки также включает поддержку указания этого атрибута в виде строкового аргумента, который будет разрешен из этой коллекции метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData). Ниже мы указываем ассоциативную таблицу **keyword\_author**, разделяющую коллекцию метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), связанную с нашей декларативной базой и ее реестром [registry](../konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl). Затем мы можем обратиться к этой таблице [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) по имени в параметре [relationship.secondary](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary):

```python
keyword_author = Table(
    'keyword_author', Base.metadata,
    Column('author_id', Integer, ForeignKey('authors.id')),
    Column('keyword_id', Integer, ForeignKey('keywords.id'))
    )

class Author(Base):
    __tablename__ = 'authors'
    id = Column(Integer, primary_key=True)
    keywords = relationship("Keyword", secondary="keyword_author")
```

Дополнительные сведения об отношениях «многие ко многим» см. в разделе «[Многие ко многим](bazovye-shablony-relationship.md#mnogie-ko-mnogim-many-to-many)».
