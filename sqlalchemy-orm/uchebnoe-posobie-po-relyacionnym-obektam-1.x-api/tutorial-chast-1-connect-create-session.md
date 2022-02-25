# Tutorial часть 1 (connect, create, session)

## Учебное пособие по реляционным объектам (1.x API)

{% hint style="info" %}
**Об этом документе**

В этом руководстве рассматривается хорошо известный API SQLAlchemy ORM, который используется уже много лет. Начиная с SQLAlchemy 1.4, существует два различных стиля использования ORM, известных как [стиль 1.x](https://docs.sqlalchemy.org/en/14/glossary.html#term-1.x-style) и [стиль 2.0](https://docs.sqlalchemy.org/en/14/glossary.html#term-2.0-style), последний из которых вносит широкий спектр изменений, наиболее заметных в том, как создаются и выполняются запросы ORM.

Планируется, что в SQLAlchemy 2.0 стиль использования ORM 1.x будет считаться устаревшим и больше не будет фигурировать в документации, а многие его аспекты будут удалены. Тем не менее, самый центральный элемент использования ORM в [стиле 1.x](https://docs.sqlalchemy.org/en/14/glossary.html#term-1.x-style), объект **Query**, по-прежнему останется доступным для долгосрочных устаревших вариантов использования.

Это руководство применимо для пользователей, которые хотят узнать, как SQLAlchemy используется в течение многих лет, особенно для тех пользователей, которые работают с существующими приложениями или связанными учебными материалами в стиле 1.x.

Введение в SQLAlchemy с новой точки зрения 1.4/2.0 см. в [учебнике по SQLAlchemy 1.4/2.0](https://docs.sqlalchemy.org/en/14/tutorial/index.html#unified-tutorial).
{% endhint %}

{% hint style="info" %}
**См. также:**

[ORM Query is internally unified with select, update, delete; 2.0 style execution available](https://docs.sqlalchemy.org/en/14/changelog/migration\_14.html#change-5159)

[Migrating to SQLAlchemy 2.0](https://docs.sqlalchemy.org/en/14/changelog/migration\_20.html)

[SQLAlchemy 1.4 / 2.0 Tutorial](https://docs.sqlalchemy.org/en/14/tutorial/index.html#unified-tutorial)
{% endhint %}

**SQLAlchemy Object Relational Mapper** представляет метод связывания пользовательских классов Python с таблицами базы данных и экземпляров этих классов (объектов) со строками в соответствующих таблицах. Он включает в себя систему, которая прозрачно синхронизирует все изменения состояния между объектами и связанными с ними строками, называемую [unit of work](https://docs.sqlalchemy.org/en/14/glossary.html#term-unit-of-work), а также систему для выражения запросов к базе данных в терминах определенных пользователем классов и их определенных отношений между собой.

ORM отличается от языка выражений SQLAlchemy, на основе которого построена ORM. В то время как язык выражений SQL, представленный в учебнике по языку выражений SQL (1.x API), представляет собой систему представления примитивных конструкций реляционной базы данных напрямую, без мнения, ORM представляет собой высокоуровневый и абстрактный шаблон использования, который сам по себе является пример прикладного использования языка выражений.

Хотя шаблоны использования ORM и языка выражений частично совпадают, сходство более поверхностно, чем может показаться на первый взгляд. Один из них подходит к структуре и содержанию данных с точки зрения определяемой пользователем [domain model](https://docs.sqlalchemy.org/en/14/glossary.html#term-domain-model), которая прозрачно сохраняется и обновляется из лежащей в ее основе модели хранения. Другой подходит к этому с точки зрения литеральной схемы и представлений выражений SQL, которые явно составлены в сообщения, потребляемые базой данных по отдельности.

Успешное приложение может быть создано исключительно с использованием Object Relational Mapper. В сложных ситуациях приложение, созданное с помощью ORM, может время от времени использовать язык выражений непосредственно в определенных областях, где требуются определенные взаимодействия с базой данных.

Следующий учебник представлен в формате doctest, что означает, что каждая строка `>>>` представляет что-то, что вы можете ввести в командной строке Python, а следующий текст представляет собой ожидаемое возвращаемое значение.

## Проверка версии

Быстрая проверка, чтобы убедиться, что мы используем по крайней мере **версию 1.4** SQLAlchemy:

```python
>>> import sqlalchemy
>>> sqlalchemy.__version__ 
1.4.0
```

## Соединение

В этом руководстве мы будем использовать базу данных **SQLite** только в памяти. Для подключения используем **create\_engine()**:

```python
>>> from sqlalchemy import create_engine
>>> engine = create_engine('sqlite:///:memory:', echo=True)
```

Флаг _**echo**_ — это ярлык для настройки ведения журнала SQLAlchemy, который выполняется с помощью стандартного модуля ведения журнала Python. Если он включен, мы увидим весь сгенерированный SQL. Если вы работаете с этим учебным пособием и хотите, чтобы выходных данных было меньше, установите для него значение `False`. Этот учебник отформатирует SQL за всплывающим окном, чтобы он не мешал нам; просто щелкните ссылку `«SQL»`, чтобы увидеть, что генерируется.

Возвращаемое значение **create\_engine()** является экземпляром **Engine** и представляет основной интерфейс к базе данных, адаптированный через диалект, который обрабатывает детали базы данных и используемого [DBAPI](https://docs.sqlalchemy.org/en/14/glossary.html#term-DBAPI). В этом случае SQLite [dialect](https://docs.sqlalchemy.org/en/14/glossary.html#term-dialect) будет интерпретировать инструкции встроенному в Python модулю `sqlite3`.

{% hint style="info" %}
**Ленивое соединение**

**Engine**, впервые возвращенный функцией **create\_engine()**, еще не пытался подключиться к базе данных; это происходит только в первый раз, когда его просят выполнить задачу для базы данных.
{% endhint %}

При первом вызове такого метода, как **Engine.execute()** или **Engine.connect()**, **Engine** устанавливает реальное соединение [DBAPI](https://docs.sqlalchemy.org/en/14/glossary.html#term-DBAPI) с базой данных, которое затем используется для отправки SQL. При использовании ORM мы обычно не используем **Engine** сразу после его создания; вместо этого он используется ORM за кулисами, как мы вскоре увидим.

{% hint style="info" %}
Смотри также:

[Database Urls](https://docs.sqlalchemy.org/en/14/core/engines.html#database-urls) — включает примеры подключения **create\_engine()** к нескольким типам баз данных со ссылками на дополнительную информацию.
{% endhint %}

## Объявление сопоставления

При использовании ORM процесс настройки начинается с описания таблиц базы данных, с которыми мы будем иметь дело, а затем с определения наших собственных классов, которые будут сопоставлены с этими таблицами. В современном SQLAlchemy эти две задачи обычно выполняются вместе с использованием системы, известной как декларативные расширения ([Declarative Extension](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html)), которая позволяет нам создавать классы, включающие директивы для описания фактической таблицы базы данных, с которой они будут сопоставлены.

Классы, отображаемые с помощью декларативной системы, определяются в терминах базового класса, который поддерживает каталог классов и таблиц, относящихся к этой базе — он известен как декларативный базовый класс (**declarative base class**). Наше приложение обычно будет иметь только один экземпляр этой базы в обычно импортируемом модуле. Мы создаем базовый класс с помощью функции **declarative\_base()** следующим образом:

```python
>>> from sqlalchemy.orm import declarative_base

>>> Base = declarative_base()
```

Теперь, когда у нас есть база **"base"**, мы можем определить любое количество сопоставленных классов в ее терминах. Мы начнем только с одной таблицы с именем _**users**_, в которой будут храниться записи для конечных пользователей, использующих наше приложение. Новый класс с именем **User** будет классом, которому мы сопоставляем эту таблицу. Внутри класса мы определяем детали таблицы, на которую мы будем отображать, в первую очередь имя таблицы, а также имена и типы данных столбцов:

```python
>>> from sqlalchemy import Column, Integer, String
>>> class User(Base):
...     __tablename__ = 'users'
...
...     id = Column(Integer, primary_key=True)
...     name = Column(String)
...     fullname = Column(String)
...     nickname = Column(String)
...
...     def __repr__(self):
...        return "<User(name='%s', fullname='%s', nickname='%s')>" % (
...                             self.name, self.fullname, self.nickname)
```

{% hint style="info" %}
**Подсказка:**

Класс **User** определяет метод `__repr__()`, но обратите внимание, что это **необязательно**; мы реализуем его только в этом руководстве, чтобы наши примеры отображали красиво отформатированные объекты **User**.
{% endhint %}

Классу, использующему **Declarative**, как минимум нужен атрибут `__tablename__` и по крайней мере один столбец **Column**, который является частью первичного ключа (1). SQLAlchemy никогда не делает никаких предположений о таблице, на которую ссылается класс, включая отсутствие встроенных соглашений об именах, типах данных или ограничениях. Но это не означает, что требуется шаблон; вместо этого вам рекомендуется создавать свои собственные автоматические соглашения, используя вспомогательные функции и классы примесей, которые подробно описаны в разделе [Примеси и пользовательские базовые классы](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/mixins.html#declarative-mixins).

Когда наш класс создан, **Declarative** заменяет все объекты **Column** специальными средствами доступа Python, известными как дескрипторы; это процесс, известный как инструментирование [instrumentation](https://docs.sqlalchemy.org/en/14/glossary.html#term-instrumentation). «Инструментально» сопоставленный класс предоставит нам средства для обращения к нашей таблице в контексте SQL, а также для сохранения и загрузки значений столбцов из базы данных.

Помимо того, что процесс сопоставления делает с нашим классом, в остальном класс остается в основном обычным классом Python, для которого мы можем определить любое количество обычных атрибутов и методов, необходимых нашему приложению.

{% hint style="info" %}
**Сноска (1)**

Сведения о том, почему требуется первичный ключ, см. в разделе [Как сопоставить таблицу, не имеющую первичного ключа?](https://docs.sqlalchemy.org/en/14/faq/ormconfiguration.html#faq-mapper-primary-key)
{% endhint %}

## Создание схемы

С нашим классом **User**, созданным с помощью декларативной системы, мы определили информацию о нашей таблице, известную как метаданные таблицы [table metadata](https://docs.sqlalchemy.org/en/14/glossary.html#term-table-metadata). Объект, используемый SQLAlchemy для представления этой информации для конкретной таблицы, называется объектом [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), и здесь **Declarative** создал его для нас. Мы можем увидеть этот объект, проверив атрибут `__table__`:

```python
>>> User.__table__ 
Table('users', MetaData(),
            Column('id', Integer(), table=<users>, primary_key=True, nullable=False),
            Column('name', String(), table=<users>),
            Column('fullname', String(), table=<users>),
            Column('nickname', String(), table=<users>), schema=None)
```

{% hint style="info" %}
**Классическое отображение**

Декларативная система, хотя и настоятельно рекомендуется, не требуется для использования ORM SQLAlchemy. Вне **Declarative** любой простой класс Python может быть сопоставлен с любой таблицей [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) с помощью функции [mapper()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper) напрямую; это менее распространенное использование описано в [Imperative (также известном как Classical) Mappings](https://docs.sqlalchemy.org/en/14/orm/mapping\_styles.html#classical-mapping).
{% endhint %}

Когда мы объявили наш класс, **Declarative** использовал метакласс Python для выполнения дополнительных действий после завершения объявления класса; на этом этапе он затем создал объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) в соответствии с нашими спецификациями и связал его с классом, создав объект [Mapper](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper). Этот объект является закулисным объектом, с которым нам обычно не нужно иметь дело напрямую (хотя он может предоставить много информации о нашем отображении, когда нам это нужно).

Объект [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) является членом более крупной коллекции, известной как [MetaData](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData). При использовании **Declarative** этот объект доступен с использованием атрибута `.metadata` нашего декларативного базового класса.

Метаданные [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) — это реестр [registry](https://docs.sqlalchemy.org/en/14/glossary.html#term-registry), который включает в себя возможность отправки в базу данных ограниченного набора команд генерации схемы. Поскольку в нашей базе данных SQLite на самом деле нет таблицы пользователей _**users**_, мы можем использовать [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) для выдачи операторов **CREATE TABLE** в базу данных для всех таблиц, которые еще не существуют. Ниже мы вызываем метод [MetaData.create\_all()](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData.create\_all), передавая наш [Engine](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Engine) в качестве источника подключения к базе данных. Мы увидим, что сначала выдаются специальные команды для проверки наличия таблицы пользователей _**users**_, а затем фактическая инструкция **CREATE TABLE**:

```python
>>> Base.metadata.create_all(engine)
BEGIN...
CREATE TABLE users (
    id INTEGER NOT NULL,
    name VARCHAR,
    fullname VARCHAR,
    nickname VARCHAR,
    PRIMARY KEY (id)
)
[...] ()
COMMIT
```

{% hint style="info" %}
**Минимальные описания таблиц против полных описаний**

Пользователи, знакомые с синтаксисом **CREATE TABLE**, могут заметить, что столбцы **VARCHAR** были сгенерированы без указания длины; в SQLite и PostgreSQL это допустимый тип данных, но в других он не разрешен. Поэтому, если вы запускаете это руководство в одной из этих баз данных и хотите использовать SQLAlchemy для выполнения команды **CREATE TABLE**, для типа [String](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.String) может быть указана `«length»`, как показано ниже:

```python
Column(String(50))
```

Поле длины в [String](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.String), а также аналогичные поля точности/масштабы, доступные в [Integer](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Integer), [Numeri](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Numeric)c и т. д., не используются SQLAlchemy, кроме как при создании таблиц.

Кроме того, **Firebird** и **Oracle** требуют последовательности для генерации новых идентификаторов первичного ключа, а SQLAlchemy не создает и не предполагает их без инструкций. Для этого вы используете конструкцию [Sequence](https://docs.sqlalchemy.org/en/14/core/defaults.html#sqlalchemy.schema.Sequence):

```python
from sqlalchemy import Sequence
Column(Integer, Sequence('user_id_seq'), primary_key=True)
```

Таким образом, полная, надежная таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), сгенерированная с помощью нашего декларативного отображения, выглядит следующим образом:

```python
class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, Sequence('user_id_seq'), primary_key=True)
    name = Column(String(50))
    fullname = Column(String(50))
    nickname = Column(String(50))

    def __repr__(self):
        return "<User(name='%s', fullname='%s', nickname='%s')>" % (
                                self.name, self.fullname, self.nickname)
```

Мы включили это более подробное определение таблицы отдельно, чтобы подчеркнуть разницу между минимальной конструкцией, предназначенной в первую очередь только для использования в Python, и конструкцией, которая будет использоваться для выдачи операторов **CREATE TABLE** в определенном наборе серверных частей с более строгими требованиями.
{% endhint %}

## Создание экземпляра сопоставленного класса Mapped Class

Когда сопоставления завершены, давайте теперь создадим и проверим объект **User**:

```python
>>> ed_user = User(name='ed', fullname='Ed Jones', nickname='edsnickname')
>>> ed_user.name
'ed'
>>> ed_user.nickname
'edsnickname'
>>> str(ed_user.id)
'None'
```

{% hint style="info" %}
**Метод `__init__()`**

Наш класс **User**, определенный с помощью декларативной системы, снабжен конструктором (например, методом **\_\_init\_\_()**), который автоматически принимает имена ключевых слов, соответствующие сопоставленным нами столбцам. Мы можем определить любой явный метод **\_\_init\_\_()**, который мы предпочитаем в нашем классе, который переопределит метод по умолчанию, предоставленный **Declarative**.
{% endhint %}

Несмотря на то, что мы не указали его в конструкторе, атрибут _**id**_ по-прежнему выдает значение `None` при доступе к нему (в отличие от обычного поведения Python, вызывающего **AttributeError** для неопределенного атрибута). Инструментарий SQLAlchemy ([instrumentation](https://docs.sqlalchemy.org/en/14/glossary.html#term-instrumentation)) обычно создает это значение по умолчанию для атрибутов сопоставления столбцов при первом доступе. Для тех атрибутов, которым мы фактически присвоили значение, система инструментирования отслеживает эти назначения для использования в возможном операторе **INSERT**, который будет отправлен в базу данных.

## Создание сессии

Теперь мы готовы начать общение с базой данных. Обработчик ("handle") ORM для базы данных — это [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session). Когда мы впервые настраиваем приложение, на том же уровне, что и наш оператор [create\_engine()](https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create\_engine), мы определяем класс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), который будет служить фабрикой для новых объектов [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session):

```python
>>> from sqlalchemy.orm import sessionmaker
>>> Session = sessionmaker(bind=engine)
```

В случае, когда ваше приложение еще не имеет [Engine](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Engine), когда вы определяете свои объекты уровня модуля, просто настройте его следующим образом:

```python
>>> Session = sessionmaker()
```

Позже, когда вы создадите свой движок с помощью [create\_engine()](https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create\_engine), подключите его к сеансу [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) с помощью [sessionmaker.configure()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.sessionmaker.configure):

```python
>>> Session.configure(bind=engine)  # как только движок engine будет доступен
```

{% hint style="info" %}
**Шаблоны жизненного цикла сеанса**

Вопрос о том, когда создавать сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), во многом зависит от того, какое приложение создается. Имейте в виду, что сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) — это просто рабочая область для ваших объектов, локальная для определенного соединения с базой данных — если вы думаете о потоке приложения как о госте на званом обеде, сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) — это тарелка гостя, а объекты, которые он содержит, — это еда. (а база данных… кухня?)! Дополнительные сведения по этой теме доступны в разделе [Когда создавать сеанс, когда его фиксировать и когда его закрывать?](https://docs.sqlalchemy.org/en/14/orm/session\_basics.html#session-faq-whentocreate)
{% endhint %}

Этот специально созданный класс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) создаст новые объекты [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), привязанные к нашей базе данных. Другие транзакционные характеристики также могут быть определены при вызове создателя сеанса [sessionmaker](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.sessionmaker); они описаны в следующей главе. Затем, всякий раз, когда вам нужно поговорить с базой данных, вы создаете экземпляр [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session):

```python
>>> session = Session()
```

Вышеуказанный сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) связан с нашим движком [Engine](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Engine) с поддержкой SQLite, но он еще не открыл никаких соединений. При первом использовании он извлекает соединение из пула соединений, поддерживаемого движком [Engine](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Engine), и удерживает его до тех пор, пока мы не зафиксируем все изменения и/или не закроем объект сеанса.
