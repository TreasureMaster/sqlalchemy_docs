# Работа с транзакциями и DBAPI

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Эта страница является частью [учебника по SQLAlchemy 1.4/2.0](./).

Предыдущий: [Установление соединения — Engine](ustanovlenie-soedineniya-engine.md) | Далее: [Работа с метаданными базы данных](rabota-s-metadannymi-bazy-dannykh.md)
{% endhint %}

Теперь, когда объект [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine) готов к работе, мы можем приступить к погружению в базовую работу [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine) и его основные интерактивные конечные точки, [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection) и [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result). Мы дополнительно представим [фасад](https://docs.sqlalchemy.org/en/14/glossary.html#term-facade) ORM для этих объектов, известный как [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session).

{% hint style="success" %}
**Примечание для читателей ORM**

При использовании ORM [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine) управляется другим объектом, называемым сеансом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session). Сессия в современной SQLAlchemy подчеркивает транзакционный и SQL-шаблон выполнения, который в значительной степени идентичен шаблону [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection), обсуждаемому ниже, поэтому, хотя этот подраздел ориентирован на Core, все концепции здесь по существу также относятся к использованию ORM и рекомендуется для всех, кто изучает ORM. Шаблон выполнения, используемый [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection), будет отличаться от шаблона [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) в конце этого раздела.
{% endhint %}

Поскольку нам еще предстоит представить язык выражений SQLAlchemy, который является основной функцией SQLAlchemy, мы будем использовать одну простую конструкцию в этом пакете, называемую конструкцией [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text), которая позволяет нам писать операторы SQL как **текстовый SQL**. Будьте уверены, что текстовый SQL в повседневном использовании SQLAlchemy является скорее исключением, чем правилом для большинства задач, хотя он всегда остается полностью доступным.

## Получение соединения

Единственная цель объекта [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine) с точки зрения пользователя — предоставить единицу подключения к базе данных, называемую [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection). При работе с Core напрямую объект [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection) — это то, как выполняется все взаимодействие с базой данных. Поскольку соединение представляет собой открытый ресурс для базы данных, мы хотим всегда ограничивать область использования этого объекта определенным контекстом, и лучший способ сделать это — использовать форму диспетчера контекста Python, также известную как [оператор with](https://docs.python.org/3/reference/compound\_stmts.html#with). Ниже мы проиллюстрируем «Hello World» с помощью текстового оператора SQL. Текстовый SQL генерируется с помощью конструкции [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text), которая будет обсуждаться более подробно позже:

```python
>>> from sqlalchemy import text

>>> with engine.connect() as conn:
...     result = conn.execute(text("select 'hello world'"))
...     print(result.all())
```

```sql
BEGIN (implicit)
select 'hello world'
[...] ()
```

```python
[('hello world',)]
```

```sql
ROLLBACK
```

В приведенном выше примере диспетчер контекста предоставил соединение с базой данных, а также обрамил операцию внутри транзакции. Поведение Python DBAPI по умолчанию включает в себя то, что транзакция всегда выполняется; когда область соединения освобождается ([released](https://docs.sqlalchemy.org/en/14/glossary.html#term-released)), выдается **ROLLBACK** для завершения транзакции. Транзакция **не фиксируется автоматически**; когда мы хотим зафиксировать данные, нам обычно нужно вызвать [Connection.commit()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.commit), как мы увидим в следующем разделе.

{% hint style="info" %}
**Совет**:

Режим «**autocommit**» доступен для особых случаев. Это обсуждается в разделе «Установка уровней изоляции транзакций, включая автоматическую фиксацию DBAPI» ([Setting Transaction Isolation Levels including DBAPI Autocommit](https://docs.sqlalchemy.org/en/14/core/connections.html#dbapi-autocommit)).
{% endhint %}

Результат нашего **SELECT** также был возвращен в объект с именем [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result), который будет обсуждаться позже, однако сейчас мы добавим, что лучше всего убедиться, что этот объект потребляется в блоке «connect» и не передается за пределы видимости нашего соединения.

## Фиксация изменений

Мы только что узнали, что соединение DBAPI **не является автоматическим**. Что, если мы хотим зафиксировать некоторые данные? Мы можем изменить наш приведенный выше пример, чтобы создать таблицу и вставить некоторые данные, а затем транзакция будет зафиксирована с использованием метода [Connection.commit()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.commit), вызываемого **внутри блока**, где мы получили объект [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection):

```python
# "commit as you go"
>>> with engine.connect() as conn:
...     conn.execute(text("CREATE TABLE some_table (x int, y int)"))
...     conn.execute(
...         text("INSERT INTO some_table (x, y) VALUES (:x, :y)"),
...         [{"x": 1, "y": 1}, {"x": 2, "y": 4}]
...     )
...     conn.commit()
```

```sql
BEGIN (implicit)
CREATE TABLE some_table (x int, y int)
[...] ()
<sqlalchemy.engine.cursor.CursorResult object at 0x...>
INSERT INTO some_table (x, y) VALUES (?, ?)
[...] ((1, 1), (2, 4))
<sqlalchemy.engine.cursor.CursorResult object at 0x...>
COMMIT
```

Выше мы создали два оператора SQL, которые обычно являются транзакционными: оператор **CREATE TABLE** \[1] и параметризованный оператор **INSERT** (приведенный выше синтаксис параметризации обсуждается несколькими разделами ниже в разделе «Отправка нескольких параметров» ([Sending Multiple Parameters](https://docs.sqlalchemy.org/en/14/tutorial/dbapi\_transactions.html#tutorial-multiple-parameters))). Поскольку мы хотим, чтобы проделанная нами работа была зафиксирована в нашем блоке, мы вызываем метод [Connection.commit()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.commit), который фиксирует транзакцию. После того, как мы вызовем этот метод внутри блока, мы можем продолжить выполнение других операторов SQL, и, если мы захотим, мы можем снова вызвать [Connection.commit()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.commit) для последующих операторов. SQLAlchemy относится к этому стилю как к **фиксации по ходу дела** (commit as you go).

Существует также другой стиль фиксации данных, который заключается в том, что мы можем заранее объявить наш блок «**connect**» блоком транзакции. Для этого режима работы мы используем метод [Engine.begin()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine.begin) для получения соединения, а не метод [Engine.connect()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine.connect). Этот метод будет управлять областью соединения [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection), а также заключать все внутри транзакции с **COMMIT** в конце, предполагая успешный блок, или **ROLLBACK** в случае возникновения исключения. Этот стиль может называться **начать один раз** (begin once):

```python
# "begin once"
>>> with engine.begin() as conn:
...     conn.execute(
...         text("INSERT INTO some_table (x, y) VALUES (:x, :y)"),
...         [{"x": 6, "y": 8}, {"x": 9, "y": 10}]
...     )
```

```sql
BEGIN (implicit)
INSERT INTO some_table (x, y) VALUES (?, ?)
[...] ((6, 8), (9, 10))
<sqlalchemy.engine.cursor.CursorResult object at 0x...>
COMMIT
```

Стиль «Начни один раз» часто предпочтительнее, поскольку он более лаконичен и указывает на намерение всего блока вперед. Однако в рамках этого руководства мы обычно будем использовать стиль «фиксация по ходу», так как он более гибкий для демонстрационных целей.

{% hint style="info" %}
**Что такое "BEGIN (implicit)"?**

Вы могли заметить строку журнала **«BEGIN (implicit)»** в начале блока транзакции. Неявный (**implicit**) здесь означает, что SQLAlchemy на самом деле _**не отправлял никакой команды**_ в базу данных; он просто считает, что это начало неявной транзакции DBAPI. Например, вы можете зарегистрировать обработчики событий для перехвата этого события.
{% endhint %}

{% hint style="info" %}
**Сноска \[ 1 ]**

[DDL](https://docs.sqlalchemy.org/en/14/glossary.html#term-DDL) относится к подмножеству SQL, которое указывает базе данных создавать, изменять или удалять конструкции уровня схемы, такие как таблицы. DDL, например **CREATE TABLE**, рекомендуется размещать в блоке транзакций, оканчивающемся на **COMMIT**, поскольку во многих базах данных используется транзакционный DDL, поэтому изменения схемы не происходят до тех пор, пока транзакция не будет зафиксирована. Однако, как мы увидим позже, мы обычно позволяем SQLAlchemy запускать последовательности DDL для нас как часть операции более высокого уровня, где нам обычно не нужно беспокоиться о **COMMIT**.
{% endhint %}

## Основы выполнения инструкций

Мы видели несколько примеров, в которых операторы SQL выполняются для базы данных с использованием метода с именем [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute) в сочетании с объектом с именем [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text) и возвратом объекта с именем [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result). В этом разделе мы более подробно проиллюстрируем механику и взаимодействие этих компонентов.

{% hint style="success" %}
Большая часть содержимого этого раздела одинаково хорошо применима к современному использованию ORM при использовании метода [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute), который работает очень похоже на метод [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute), включая то, что строки результатов ORM доставляются с использованием того же интерфейса [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result), используемого Core.
{% endhint %}

### Получение строк

Сначала мы более подробно проиллюстрируем объект [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result), используя строки, которые мы вставили ранее, выполнив текстовый оператор **SELECT** для созданной нами таблицы:

```python
>>> with engine.connect() as conn:
...     result = conn.execute(text("SELECT x, y FROM some_table"))
...     for row in result:
...         print(f"x: {row.x}  y: {row.y}")
```

```sql
BEGIN (implicit)
SELECT x, y FROM some_table
[...] ()
```

```python
x: 1  y: 1
x: 2  y: 4
x: 6  y: 8
x: 9  y: 10
```

```sql
ROLLBACK
```

Выше строка «**SELECT**», которую мы выполнили, выбрала все строки из нашей таблицы. Возвращаемый объект называется [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result) и представляет итерируемый объект строк результатов.

[Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result) имеет множество методов для выборки и преобразования строк, таких как показанный ранее метод [Result.all()](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result.all), который возвращает список всех объектов [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row). Он также реализует интерфейс итератора Python, так что мы можем напрямую перебирать коллекцию объектов [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row).

Сами объекты [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row) должны действовать как именованные кортежи Python. Ниже мы иллюстрируем различные способы доступа к строкам.

* **Tuple Assignment** - Это наиболее идиоматический стиль Python, который заключается в том, чтобы присваивать переменные каждой строке позиционно по мере их получения:

```python
result = conn.execute(text("select x, y from some_table"))

for x, y in result:
    # ...
```

* **Integer Index** - Кортежи — это последовательности Python, поэтому доступен и обычный целочисленный доступ:

```python
result = conn.execute(text("select x, y from some_table"))

  for row in result:
      x = row[0]
```

* **Attribute Name** - Поскольку это именованные кортежи Python, кортежи имеют динамические имена атрибутов, соответствующие именам каждого столбца. Эти имена обычно являются именами, которые оператор SQL присваивает столбцам в каждой строке. Хотя они обычно довольно предсказуемы и также могут контролироваться метками, в менее определенных случаях они могут зависеть от поведения, специфичного для базы данных:

```python
result = conn.execute(text("select x, y from some_table"))

for row in result:
    y = row.y

    # иллюстрация использования с Python f-strings
    print(f"Row: {row.x} {row.y}")
```

* **Mapping Access** - Для получения строк в качестве объектов сопоставления (**mapping**) Python, которые по сути являются доступной только для чтения версией интерфейса Python для общего объекта **dict**, [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result) может быть преобразован в объект [MappingResult](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.MappingResult) с использованием модификатора [Result.mappings()](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result.mappings); это объект результата, который дает объекты [RowMapping](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.RowMapping), подобные словарю, а не объекты [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row):

```python
result = conn.execute(text("select x, y from some_table"))

for dict_row in result.mappings():
    x = dict_row['x']
    y = dict_row['y']
```

### Отправка параметров

Операторы SQL обычно сопровождаются данными, которые должны быть переданы вместе с самим оператором, как мы видели ранее в примере **INSERT**. Таким образом, метод [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute) также принимает параметры, которые называются связанными параметрами ([bound parameters](https://docs.sqlalchemy.org/en/14/glossary.html#term-bound-parameters)). Элементарным примером может быть ситуация, когда мы хотим ограничить наш оператор **SELECT** только строками, которые соответствуют определенным критериям, например, строками, в которых значение «y» превышает определенное значение, которое передается функции.

Чтобы добиться этого, чтобы оператор SQL мог оставаться фиксированным и чтобы драйвер мог правильно очистить значение, мы добавляем критерий **WHERE** в наш оператор, который называет новый параметр с именем «y»; конструкция [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text) принимает их, используя формат двоеточия `«:y»`. Фактическое значение `«:y»` затем передается в качестве второго аргумента функции [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute) в виде словаря:

```python
>>> with engine.connect() as conn:
...     result = conn.execute(
...         text("SELECT x, y FROM some_table WHERE y > :y"),
...         {"y": 2}
...     )
...     for row in result:
...        print(f"x: {row.x}  y: {row.y}")
```

```sql
BEGIN (implicit)
SELECT x, y FROM some_table WHERE y > ?
[...] (2,)
```

```python
x: 2  y: 4
x: 6  y: 8
x: 9  y: 10
```

```sql
ROLLBACK
```

В зарегистрированном выводе SQL мы видим, что связанный параметр `:y` был преобразован в вопросительный знак, когда он был отправлен в базу данных SQLite. Это связано с тем, что драйвер базы данных SQLite использует формат, называемый «**стиль параметра qmark**», который является одним из шести различных форматов, разрешенных спецификацией DBAPI. SQLAlchemy абстрагирует эти форматы только в один, который является «**named**» форматом с использованием двоеточия.

{% hint style="info" %}
**Всегда используйте связанные параметры**

Как упоминалось в начале этого раздела, текстовый SQL не является обычным способом работы с SQLAlchemy. Однако при использовании текстового SQL литеральное значение Python, даже не-строки, такие как целые числа или даты, **никогда не должно напрямую преобразовываться в строку SQL**; _**всегда**_ следует использовать параметр. Это наиболее известно как способ избежать атак с помощью SQL-инъекций, когда данные ненадежны. Однако это также позволяет диалектам SQLAlchemy и/или DBAPI правильно обрабатывать входящие входные данные для серверной части. За исключением случаев использования простого текстового SQL, SQLAlchemy Core Expression API в противном случае гарантирует, что литеральные значения Python передаются в качестве связанных параметров, где это необходимо.
{% endhint %}

### Отправка нескольких параметров

В примере [с фиксацией изменений](rabota-s-tranzakciyami-i-dbapi.md#fiksaciya-izmenenii) мы выполнили оператор **INSERT**, где оказалось, что мы можем **INSERT** сразу несколько строк в базу данных. Для операторов, которые **работают с данными, но не возвращают наборы результатов**, а именно для операторов [DML](https://docs.sqlalchemy.org/en/14/glossary.html#term-DML), таких как «**INSERT**», которые не включают фразу типа «**RETURNING**», мы можем отправить несколько параметров в метод [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute), передав список словарей вместо одного словаря, что позволяет вызывать один оператор SQL для каждого набора параметров отдельно:

```python
>>> with engine.connect() as conn:
...     conn.execute(
...         text("INSERT INTO some_table (x, y) VALUES (:x, :y)"),
...         [{"x": 11, "y": 12}, {"x": 13, "y": 14}]
...     )
...     conn.commit()
```

```sql
BEGIN (implicit)
INSERT INTO some_table (x, y) VALUES (?, ?)
[...] ((11, 12), (13, 14))
<sqlalchemy.engine.cursor.CursorResult object at 0x...>
COMMIT
```

За кулисами объекты [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection) используют функцию DBAPI, известную как [cursor.executemany()](https://www.python.org/dev/peps/pep-0249/#id18). Этот метод выполняет эквивалентную операцию вызова данного оператора SQL для каждого набора параметров в отдельности. DBAPI может оптимизировать эту операцию различными способами, используя подготовленные операторы или в некоторых случаях объединяя наборы параметров в один оператор SQL. Некоторые диалекты SQLAlchemy могут также использовать альтернативные API для этого случая, такие как диалект psycopg2 для PostgreSQL ([psycopg2 dialect for PostgreSQL](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html#postgresql-psycopg2)), который использует более производительные API для этого случая использования.

{% hint style="info" %}
**Совет**:

вы могли заметить, что этот раздел не помечен как **концепция ORM**. Это связано с тем, что вариант использования «**multiple parameters**» _**обычно**_ используется для операторов **INSERT**, которые при использовании ORM вызываются другим способом. Несколько параметров также могут использоваться с операторами **UPDATE** и **DELETE** для создания отдельных операций **UPDATE/DELETE** для каждой строки, однако, опять же, при использовании ORM существует другой метод, обычно используемый для обновления или удаления многих отдельных строк по отдельности.
{% endhint %}

### Связывание параметров с оператором

Два предыдущих случая иллюстрируют ряд параметров, передаваемых вместе с оператором SQL. Для выполнения операторов с одним параметром использование параметров в SQLAlchemy на самом деле чаще всего осуществляется путем связывания параметров (**binding**) с самим оператором, что является основной функцией языка выражений SQL и позволяет создавать запросы, которые могут быть составлены естественным образом, при этом во всех случаях используется параметризация. Эта концепция будет обсуждаться более подробно в следующих разделах; для краткого предварительного просмотра сама конструкция [text()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.text), являющаяся частью языка выражений SQL, поддерживает эту функцию с помощью метода [TextClause.bindparams()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.TextClause.bindparams); это [генеративный](https://docs.sqlalchemy.org/en/14/glossary.html#term-generative) метод, который возвращает новую копию конструкции SQL с добавленным дополнительным состоянием, в данном случае значения параметров, которые мы хотим передать:

```python
>>> stmt = text("SELECT x, y FROM some_table WHERE y > :y ORDER BY x, y").bindparams(y=6)
>>> with engine.connect() as conn:
...     result = conn.execute(stmt)
...     for row in result:
...        print(f"x: {row.x}  y: {row.y}")
```

```sql
BEGIN (implicit)
SELECT x, y FROM some_table WHERE y > ? ORDER BY x, y
[...] (6,)
```

```python
x: 6  y: 8
x: 9  y: 10
x: 11  y: 12
x: 13  y: 14
```

```sql
ROLLBACK
```

Интересно отметить выше, что хотя мы передали только один аргумент, **stmt**, методу [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute), выполнение оператора иллюстрировало как строку SQL, так и отдельный кортеж параметров.

## Выполнение с сеансом ORM Session

Как упоминалось ранее, большинство шаблонов и примеров, приведенных выше, также применимы для использования с ORM, поэтому здесь мы представим это использование, чтобы по ходу обучения мы могли проиллюстрировать каждый шаблон с точки зрения совместного использования Core и ORM.

Фундаментальный интерактивный объект транзакций/базы данных при использовании ORM называется сеансом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session). В современной SQLAlchemy этот объект используется способом, очень похожим на метод [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection), и фактически, когда используется [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), он ссылается на внутреннее соединение [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection), которое он использует для генерации SQL.

Когда сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) используется с конструкциями, отличными от ORM, он проходит через операторы SQL, которые мы ему даем, и обычно не сильно отличается от того, как делает соединение [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection) напрямую, поэтому мы можем проиллюстрировать это здесь с точки зрения простых текстовых операций SQL, которым мы уже научились.

Сессия [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) имеет несколько различных шаблонов создания, но здесь мы проиллюстрируем самый простой из них, который точно отслеживает то, как используется соединение [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection), то есть его создание в диспетчере контекста:

```python
>>> from sqlalchemy.orm import Session

>>> stmt = text("SELECT x, y FROM some_table WHERE y > :y ORDER BY x, y").bindparams(y=6)
>>> with Session(engine) as session:
...     result = session.execute(stmt)
...     for row in result:
...        print(f"x: {row.x}  y: {row.y}")
```

```sql
BEGIN (implicit)
SELECT x, y FROM some_table WHERE y > ? ORDER BY x, y
[...] (6,)
```

```python
x: 6  y: 8
x: 9  y: 10
x: 11  y: 12
x: 13  y: 14
```

```sql
ROLLBACK
```

Приведенный выше пример можно сравнить с примером в предыдущем разделе «[Связывание параметров с оператором](rabota-s-tranzakciyami-i-dbapi.md#svyazyvanie-parametrov-s-operatorom)» — мы напрямую заменяем вызов `with engine.connect() as conn` на `with Session(engine) as session`, а затем используем [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute) точно так же, как и с методом [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute).

Кроме того, как и [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection), [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) имеет поведение «фиксация по ходу» с использованием метода [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit), показанного ниже с использованием текстового оператора **UPDATE** для изменения некоторых наших данных:

```python
>>> with Session(engine) as session:
...     result = session.execute(
...         text("UPDATE some_table SET y=:y WHERE x=:x"),
...         [{"x": 9, "y":11}, {"x": 13, "y": 15}]
...     )
...     session.commit()
```

```sql
BEGIN (implicit)
UPDATE some_table SET y=? WHERE x=?
[...] ((11, 9), (15, 13))
COMMIT
```

Выше мы вызвали оператор **UPDATE**, используя связанный параметр, стиль выполнения «**executemany**», введенный в «[Отправке нескольких параметров](rabota-s-tranzakciyami-i-dbapi.md#otpravka-neskolkikh-parametrov)», заканчивая блок фиксацией «**commit as you go**».

{% hint style="info" %}
**Совет**:

[Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) на самом деле не удерживает объект [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection) после завершения транзакции. Он получает новое соединение [Connection](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection) от [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine), когда в следующий раз требуется выполнение SQL для базы данных.
{% endhint %}

У [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), очевидно, гораздо больше хитростей в рукаве, однако понимание того, что у него есть метод [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute), который используется так же, как [Connection.execute()](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Connection.execute), поможет нам начать с примеров, которые последуют позже.

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел руководства: [Работа с метаданными базы данных](rabota-s-metadannymi-bazy-dannykh.md)
{% endhint %}
