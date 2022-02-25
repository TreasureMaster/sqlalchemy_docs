# Выбор строк (работа с SQL функциями)

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Эта страница является частью [учебника по SQLAlchemy 1.4/2.0](../).

Предыдущий: [Выбор строк с помощью Core и ORM](vybor-strok-s-pomoshyu-core-ili-orm.md) | Далее: [Обновление и удаление строк с помощью Core](obnovlenie-i-udalenie-strok-s-pomoshyu-core.md)
{% endhint %}

## Выбор строк (работа с SQL функциями)

Объект [func](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.func), впервые представленный ранее в этом разделе в разделе [Агрегатные функции с GROUP BY / HAVING](vybor-strok-s-pomoshyu-core-ili-orm.md#agregatnye-funkcii-s-group-by-having), служит фабрикой для создания новых объектов [Function](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.Function), которые при использовании в такой конструкции, как [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), создают отображение функции SQL, обычно состоящее из имени , некоторые скобки (хотя и не всегда) и, возможно, некоторые аргументы. Примеры типичных функций SQL включают в себя:

* функция **count()**, агрегатная функция, которая подсчитывает количество возвращенных строк:

```python
>>> print(select(func.count()).select_from(user_table))
SELECT count(*) AS count_1
FROM user_account
```

* функция **lower()**, строковая функция, которая преобразует строку в нижний регистр:

```python
>>> print(select(func.lower("A String With Much UPPERCASE")))
SELECT lower(:lower_2) AS lower_1
```

* функция **now()**, предоставляющая текущую дату и время; поскольку это общая функция, SQLAlchemy знает, как отображать ее по-разному для каждого бэкэнда, в случае SQLite с использованием функции **CURRENT\_TIMESTAMP**:

```python
>>> stmt = select(func.now())
>>> with engine.connect() as conn:
...     result = conn.execute(stmt)
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT CURRENT_TIMESTAMP AS now_1
[...] ()
[(datetime.datetime(...),)]
ROLLBACK
```

Поскольку большинство серверных частей баз данных содержат десятки, если не сотни различных функций SQL, [func](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.func) старается быть как можно более либеральным в том, что он принимает. Любое имя, доступ к которому осуществляется из этого пространства имен, автоматически считается функцией SQL, которая будет отображаться в общем виде:

```python
>>> print(select(func.some_crazy_function(user_table.c.name, 17)))
SELECT some_crazy_function(user_account.name, :some_crazy_function_2) AS some_crazy_function_1
FROM user_account
```

В то же время относительно небольшой набор чрезвычайно распространенных функций SQL, таких как [count](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.count), [now](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.now), [max](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.max), [concat](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.concat), включает в себя предварительно упакованные версии самих себя, которые обеспечивают правильную типизацию информации, а также в некоторых случаях генерацию SQL для бэкэнда. В приведенном ниже примере сравнивается генерация SQL, которая происходит для диалекта **PostgreSQL**, по сравнению с диалектом **Oracle** для функции [now](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.now):

```python
>>> from sqlalchemy.dialects import postgresql
>>> print(select(func.now()).compile(dialect=postgresql.dialect()))
SELECT now() AS now_1

>>> from sqlalchemy.dialects import oracle
>>> print(select(func.now()).compile(dialect=oracle.dialect()))
SELECT CURRENT_TIMESTAMP AS now_1 FROM DUAL
```

### Функции имеют возвращаемые типы

Поскольку функции являются выражениями столбцов, они также имеют [типы данных](https://docs.sqlalchemy.org/en/14/core/types.html) SQL, которые описывают тип данных сгенерированного выражения SQL. Мы называем эти типы здесь «**типами возврата SQL**» в отношении типа значения SQL, которое возвращается функцией в контексте выражения SQL на стороне базы данных, в отличие от «**типа возврата**» функции Python.

Доступ к возвращаемому типу SQL любой функции SQL можно получить, как правило, в целях отладки, обратившись к атрибуту [Function.type](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.Function.type):

```python
>>> func.now().type
DateTime()
```

Эти возвращаемые типы SQL важны при использовании выражения функции в контексте более крупного выражения; то есть математические операторы будут работать лучше, когда тип данных выражения будет чем-то вроде [Integer](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Integer) или [Numeric](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Numeric), средства доступа JSON для работы должны использовать такой тип, как [JSON](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.JSON). Некоторые классы функций возвращают целые строки вместо значений столбцов, когда необходимо обратиться к конкретным столбцам; такие функции называются [табличными функциями](vybor-strok-rabota-s-sql-funkciyami.md#tablichnye-funkcii).

Тип возврата функции SQL также может иметь значение при выполнении инструкции и возврате строк в тех случаях, когда SQLAlchemy должна применять обработку набора результатов. Ярким примером этого являются функции, связанные с датой, в SQLite, где [DateTime](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.DateTime) и связанные типы данных SQLAlchemy берут на себя роль преобразования строковых значений в объекты Python `datetime()` по мере получения строк результатов.

Чтобы применить определенный тип к создаваемой нами функции, мы передаем его с помощью параметра [Function.type\_](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.Function.params.type\_); аргумент типа может быть либо классом [TypeEngine](https://docs.sqlalchemy.org/en/14/core/type\_api.html#sqlalchemy.types.TypeEngine), либо экземпляром. В приведенном ниже примере мы передаем класс [JSON](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.JSON) для создания функции PostgreSQL `json_object()`, отмечая, что возвращаемый тип SQL будет иметь тип JSON:

```python
>>> from sqlalchemy import JSON
>>> function_expr = func.json_object('{a, 1, b, "def", c, 3.5}', type_=JSON)
```

Создавая нашу функцию JSON с типом данных [JSON](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.JSON), объект выражения SQL приобретает функции, связанные с JSON, такие как доступ к элементам:

```python
>>> stmt = select(function_expr["def"])
>>> print(stmt)
SELECT json_object(:json_object_1)[:json_object_2] AS anon_1
```

### Встроенные функции имеют предварительно настроенные типы возврата

Для общих агрегатных функций, таких как [count](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.count), [max](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.max), [min](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.min), а также очень небольшого числа функций даты, таких как [now](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.now), и строковых функций, таких как [concat](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.concat), тип возвращаемого значения SQL настраивается соответствующим образом, иногда в зависимости от использования. Функция [max](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.max) и аналогичные функции фильтрации агрегатов установят тип возвращаемого значения SQL на основе заданного аргумента:

```python
>>> m1 = func.max(Column("some_int", Integer))
>>> m1.type
Integer()

>>> m2 = func.max(Column("some_str", String))
>>> m2.type
String()
```

Функции даты и времени обычно соответствуют выражениям SQL, описанным [DateTime](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.DateTime), [Date](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Date) или [Time](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Time):

```python
>>> func.now().type
DateTime()
>>> func.current_date().type
Date()
```

Известная строковая функция, такая как [concat](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.concat), будет знать, что выражение SQL будет иметь тип [String](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.String):

```python
>>> func.concat("x", "y").type
String()
```

Однако для подавляющего большинства функций SQL SQLAlchemy не имеет их явным образом в своем очень небольшом списке известных функций. Например, хотя обычно нет проблем с использованием функций SQL **func.lower()** и **func.upper()** для преобразования регистра строк, SQLAlchemy на самом деле не знает об этих функциях, поэтому они имеют тип возвращаемого значения SQL **«null»**:

```python
>>> func.upper("lowercase").type
NullType()
```

Для простых функций, таких как **upper** и **lower**, проблема обычно незначительна, так как строковые значения могут быть получены из базы данных без какой-либо специальной обработки типов на стороне SQLAlchemy, а правила приведения типов SQLAlchemy также часто могут правильно угадать намерение; например, оператор Python **+** будет правильно интерпретирован как оператор конкатенации строк на основе просмотра обеих сторон выражения:

```python
>>> print(select(func.upper("lowercase") + " suffix"))
SELECT upper(:upper_1) || :upper_2 AS anon_1
```

В целом сценарий, в котором, вероятно, необходим параметр [Function.type\_](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.Function.params.type\_), выглядит следующим образом:

* эта функция еще не является встроенной функцией SQLAlchemy; в этом можно убедиться, создав функцию и наблюдая за атрибутом [Function.type](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.Function.type), то есть:

```python
>>> func.count().type
Integer()
```

Против:

```python
>>> func.json_object('{"a", "b"}').type
NullType()
```

* Необходима поддержка выражений с учетом функций; чаще всего это относится к специальным операторам, связанным с типами данных, такими как [JSON](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.JSON) или [ARRAY](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.ARRAY).
* Требуется обработка результирующего значения, которое может включать такие типы, как **DateTime**, [Boolean](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Boolean), [Enum](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Enum), или специальные типы данных, такие как [JSON](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.JSON), [ARRAY](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.ARRAY).

### Использование оконных функций

**Оконная функция** — это специальное использование агрегатной функции SQL, которая вычисляет совокупное значение по строкам, возвращаемым в группе, по мере обработки отдельных строк результата. В то время как функция, такая как **MAX()**, даст вам самое высокое значение столбца в наборе строк, использование той же функции, что и оконная функция, даст вам самое высокое значение для **каждой** строки, _**начиная с этой строки**_.

В SQL оконные функции позволяют указать строки, к которым функция должна применяться, значение «раздела» (_partition_), которое рассматривает окно по различным подмножествам строк, и выражение «**order by**», которое важно указывает порядок, в котором строки должны применяться к агрегатной функции.

В SQLAlchemy все функции SQL, сгенерированные пространством имен [func](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.func), включают метод [FunctionElement.over()](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.over), который предоставляет оконную функцию или синтаксис «**OVER**»; полученная конструкция является конструкцией [Over](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.Over).

Обычной функцией, используемой с оконными функциями, является функция **row\_number()**, которая просто подсчитывает строки. Мы можем разделить это количество строк по имени пользователя, чтобы пронумеровать адреса электронной почты отдельных пользователей:

```python
>>> stmt = select(
...     func.row_number().over(partition_by=user_table.c.name),
...     user_table.c.name,
...     address_table.c.email_address
... ).select_from(user_table).join(address_table)
>>> with engine.connect() as conn:  
...     result = conn.execute(stmt)
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT row_number() OVER (PARTITION BY user_account.name) AS anon_1,
user_account.name, address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
[...] ()
```

```python
[(1, 'sandy', 'sandy@sqlalchemy.org'),
 (2, 'sandy', 'sandy@squirrelpower.org'),
 (1, 'spongebob', 'spongebob@sqlalchemy.org')]
```

```sql
ROLLBACK
```

Выше параметр [FunctionElement.over.partition\_by](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.over.params.partition\_by) используется, чтобы предложение **PARTITION BY** отображалось в предложении **OVER**. Мы также можем использовать предложение **ORDER BY** с помощью [FunctionElement.over.order\_by](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.over.params.order\_by):

```python
>>> stmt = select(
...     func.count().over(order_by=user_table.c.name),
...     user_table.c.name,
...     address_table.c.email_address).select_from(user_table).join(address_table)
>>> with engine.connect() as conn:  
...     result = conn.execute(stmt)
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT count(*) OVER (ORDER BY user_account.name) AS anon_1,
user_account.name, address.email_address
FROM user_account JOIN address ON user_account.id = address.user_id
[...] ()
```

```python
[(2, 'sandy', 'sandy@sqlalchemy.org'),
 (2, 'sandy', 'sandy@squirrelpower.org'),
 (3, 'spongebob', 'spongebob@sqlalchemy.org')]
```

```sql
ROLLBACK
```

Другие варианты оконных функций включают использование диапазонов; см. [over()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.over) для получения дополнительных примеров.

{% hint style="info" %}
Совет:

Важно отметить, что метод [FunctionElement.over()](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.over) применяется только к тем функциям SQL, которые на самом деле являются агрегатными функциями; в то время как конструкция [Over](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.Over) будет успешно отображать себя для любой заданной функции SQL, база данных отклонит выражение, если сама функция не является агрегатной функцией SQL.
{% endhint %}

### Специальные модификаторы WITHIN GROUP, FILTER

Синтаксис SQL «**WITHIN GROUP**» используется в сочетании с агрегатной функцией «**ordered set**» или «**hypothetical set**». Общие функции «**ordered set**» включают в себя **percentile\_cont()** и **rank()**. SQLAlchemy включает встроенные реализации [rank](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.rank), [dense\_rank](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.dense\_rank), [mode](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.mode), [percentile\_cont](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.percentile\_cont) и [percentile\_disk](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.percentile\_disc), которые включают метод [FunctionElement.within\_group()](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.within\_group):

```python
>>> print(
...     func.unnest(
...         func.percentile_disc([0.25,0.5,0.75,1]).within_group(user_table.c.name)
...     )
... )
unnest(percentile_disc(:percentile_disc_1) WITHIN GROUP (ORDER BY user_account.name))
```

«**FILTER**» поддерживается некоторыми бэкендами для ограничения диапазона агрегатной функции определенным подмножеством строк по сравнению с общим диапазоном возвращаемых строк, доступных с помощью метода [FunctionElement.filter()](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.filter):

```python
>>> stmt = select(
...     func.count(address_table.c.email_address).filter(user_table.c.name == 'sandy'),
...     func.count(address_table.c.email_address).filter(user_table.c.name == 'spongebob')
... ).select_from(user_table).join(address_table)
>>> with engine.connect() as conn:  
...     result = conn.execute(stmt)
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT count(address.email_address) FILTER (WHERE user_account.name = ?) AS anon_1,
count(address.email_address) FILTER (WHERE user_account.name = ?) AS anon_2
FROM user_account JOIN address ON user_account.id = address.user_id
[...] ('sandy', 'spongebob')
```

```python
[(2, 1)]
```

```sql
ROLLBACK
```

### Функции с табличным значением

Табличные функции SQL поддерживают скалярное представление, содержащее именованные вложенные элементы. Часто используемая для функций, ориентированных на JSON и ARRAY, а также таких функций, как **generate\_series()**, функция с табличным значением указывается в предложении **FROM** и затем упоминается как таблица, а иногда даже как столбец. Функции этой формы широко используются в базе данных **PostgreSQL**, однако некоторые формы функций с табличным значением также поддерживаются **SQLite**, **Oracle** и **SQL Server**.

{% hint style="info" %}
Смотри также:

[Table values, Table and Column valued functions, Row and Tuple objects](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html#postgresql-table-valued-overview) — в документации [PostgreSQL](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html).

В то время как многие базы данных поддерживают табличное значение и другие специальные формы, **PostgreSQL**, как правило, находится там, где существует наибольший спрос на эти функции. См. этот раздел для получения дополнительных примеров синтаксиса **PostgreSQL**, а также дополнительных функций.
{% endhint %}

SQLAlchemy предоставляет метод [FunctionElement.table\_valued()](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.table\_valued) в качестве базовой конструкции «функция с табличным значением», которая преобразует объект [func](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.func) в предложение **FROM**, содержащее ряд именованных столбцов на основе строковых имен, передаваемых позиционно. Это возвращает объект [TableValuedAlias](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.TableValuedAlias), который представляет собой конструкцию [Alias](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Alias) с включенной функцией, которую можно использовать в качестве любого другого предложения **FROM**, как описано в разделе [Использование псевдонимов](vybor-strok-s-pomoshyu-core-ili-orm.md#ispolzovanie-psevdonimov-alias). Ниже мы проиллюстрируем функцию **json\_each()**, которая, хотя и распространена в **PostgreSQL**, также поддерживается современными версиями SQLite:

```python
>>> onetwothree = func.json_each('["one", "two", "three"]').table_valued("value")
>>> stmt = select(onetwothree).where(onetwothree.c.value.in_(["two", "three"]))
>>> with engine.connect() as conn:  
...     result = conn.execute(stmt)
...     print(result.all())
```

```sql
BEGIN (implicit)
SELECT anon_1.value
FROM json_each(?) AS anon_1
WHERE anon_1.value IN (?, ?)
[...] ('["one", "two", "three"]', 'two', 'three')
```

```python
[('two',), ('three',)]
```

```sql
ROLLBACK
```

Выше мы использовали функцию JSON **json\_each()**, поддерживаемую **SQLite** и **PostgreSQL**, для создания табличного выражения с одним столбцом, указанным как **value**, а затем выбрали две из трех его строк.

{% hint style="info" %}
Смотри также:

[Table-Valued Functions](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html#postgresql-table-valued) — в документации [PostgreSQL](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html) — в этом разделе подробно описаны дополнительные синтаксисы, такие как специальные производные столбцов и «**WITH ORDINALITY**», которые, как известно, работают с **PostgreSQL**.
{% endhint %}

### Функции со значениями столбца — функция с табличным значением как скалярный столбец

Специальный синтаксис, поддерживаемый **PostgreSQL** и **Oracle**, заключается в обращении к функции в предложении **FROM**, которая затем доставляется как один столбец в предложении столбцов оператора **SELECT** или в другом контексте выражения столбца. **PostgreSQL** отлично использует этот синтаксис для таких функций, как `json_array_elements()`, `json_object_keys()`, `json_each_text()`, `json_each()` и т. д.

SQLAlchemy называет это функцией «**column valued**» и доступно, применяя модификатор [FunctionElement.column\_valued()](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.FunctionElement.column\_valued) к конструкции [Function](https://docs.sqlalchemy.org/en/14/core/functions.html#sqlalchemy.sql.functions.Function):

```python
>>> from sqlalchemy import select, func
>>> stmt = select(func.json_array_elements('["one", "two"]').column_valued("x"))
>>> print(stmt)
SELECT x
FROM json_array_elements(:json_array_elements_1) AS x
```

Форма «**column valued**» также поддерживается диалектом **Oracle**, где ее можно использовать для пользовательских функций SQL:

```python
>>> from sqlalchemy.dialects import oracle
>>> stmt = select(func.scalar_strings(5).column_valued("s"))
>>> print(stmt.compile(dialect=oracle.dialect()))
SELECT COLUMN_VALUE s
FROM TABLE (scalar_strings(:scalar_strings_1)) s
```

{% hint style="info" %}
Смотри также:

[Column Valued Functions](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html#postgresql-column-valued) - в документации [PostgreSQL](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html).
{% endhint %}

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел руководства: [Обновление и удаление строк с помощью Core](obnovlenie-i-udalenie-strok-s-pomoshyu-core.md)
{% endhint %}
