# Манипуляции с данными с помощью ORM

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Эта страница является частью [учебника по SQLAlchemy 1.4/2.0](./).

Предыдущий: [Обновление и удаление строк с помощью Core](rabota-s-dannymi/obnovlenie-i-udalenie-strok-s-pomoshyu-core.md) | Далее: [Работа со связанными объектами](rabota-so-svyazannymi-obektami.md)
{% endhint %}

Предыдущий раздел «[Работа с данными](rabota-s-dannymi/)» по-прежнему был сосредоточен на языке выражений SQL с точки зрения **Core**, чтобы обеспечить непрерывность основных конструкций операторов SQL. Затем в этом разделе будет описан жизненный цикл [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) и то, как он взаимодействует с этими конструкциями.

**Предварительные разделы** — часть руководства, ориентированная на **ORM**, основывается на двух предыдущих разделах этого документа, посвященных ORM:

* [Выполнение с сеансом ORM](rabota-s-tranzakciyami-i-dbapi.md#vypolnenie-s-seansom-orm-session) - рассказывает, как создать объект сеанса ORM.
* [Определение метаданных таблицы с помощью ORM](rabota-s-metadannymi-bazy-dannykh.md#opredelenie-metadannykh-tablicy-s-pomoshyu-orm) — где мы настраиваем наши ORM-сопоставления сущностей пользователя **User** и адреса **Address**.
* [Выбор сущностей и столбцов ORM](rabota-s-dannymi/vybor-strok-s-pomoshyu-core-ili-orm.md#vybor-obektov-i-stolbcov-orm) — несколько примеров того, как запускать операторы **SELECT** для таких сущностей, как **User**.

## Вставка строк с ORM

При использовании ORM объект [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) отвечает за создание конструкций [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert) и их отправку для нас в транзакции. Способ, которым мы инструктируем [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) сделать это, заключается в **добавлении** к нему записей объекта; затем сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) гарантирует, что эти новые записи будут отправлены в базу данных, когда они потребуются, используя процесс, известный как сброс **flush**.

### Экземпляры классов представляют строки (Row)

В то время как в предыдущем примере мы сгенерировали **INSERT**, используя словари Python, чтобы указать данные, которые мы хотели добавить, с ORM мы напрямую используем пользовательские классы Python, которые мы определили, еще в разделе «[Определение метаданных таблицы с помощью ORM](rabota-s-metadannymi-bazy-dannykh.md#opredelenie-metadannykh-tablicy-s-pomoshyu-orm)». На уровне класса классы **User** и **Address** служили местом для определения того, как должны выглядеть соответствующие таблицы базы данных. Эти классы также служат расширяемыми объектами данных, которые мы также используем для создания и управления строками внутри транзакции. Ниже мы создадим два объекта **User**, каждый из которых представляет потенциальную строку базы данных для **INSERT**:

```python
>>> squidward = User(name="squidward", fullname="Squidward Tentacles")
>>> krabs = User(name="ehkrabs", fullname="Eugene H. Krabs")
```

Мы можем создавать эти объекты, используя имена сопоставленных столбцов в качестве аргументов ключевого слова в конструкторе. Это возможно, поскольку класс **User** включает в себя автоматически сгенерированный конструктор **\_\_init\_\_()**, который был предоставлен сопоставлением (_mapping_) ORM, чтобы мы могли создавать каждый объект, используя имена столбцов в качестве ключей в конструкторе.

Аналогично тому, как в наших примерах Core [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert), мы не включали первичный ключ (то есть запись для столбца **id**), поскольку мы хотели бы использовать функцию автоинкрементного первичного ключа базы данных, **SQLite** в этом случае также интегрируется ORM. Значение атрибута **id** для вышеуказанных объектов, если бы мы его просмотрели, отображается как `None`:

```python
>>> squidward
User(id=None, name='squidward', fullname='Squidward Tentacles')
```

Значение `None` предоставляется SQLAlchemy, чтобы указать, что атрибут еще не имеет значения. Атрибуты, отображаемые SQLAlchemy, всегда возвращают значение в Python и не вызывают **AttributeError**, если они отсутствуют, при работе с новым объектом, которому не присвоено значение.

На данный момент говорят, что два наших объекта выше находятся в состоянии, называемом переходным ([transient](https://docs.sqlalchemy.org/en/14/glossary.html#term-transient)), — они не связаны ни с каким состоянием базы данных и еще должны быть связаны с объектом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), который может генерировать для них операторы **INSERT**.

### Добавление объектов в сеанс Session

Чтобы проиллюстрировать процесс добавления шаг за шагом, мы создадим сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) без использования контекстного менеджера (и, следовательно, мы должны убедиться, что закроем его позже!):

```python
>>> session = Session(engine)
```

Затем объекты добавляются в сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) с помощью метода [Session.add()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.add). Когда это вызывается, объекты находятся в состоянии ожидания ([pending](https://docs.sqlalchemy.org/en/14/glossary.html#term-pending)) и еще не были вставлены:

```python
>>> session.add(squidward)
>>> session.add(krabs)
```

Когда у нас есть ожидающие объекты, мы можем увидеть это состояние, просматривая коллекцию в сеансе [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) под названием [Session.new](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.new):

```python
>>> session.new
IdentitySet(
    [User(id=None, name='squidward', fullname='Squidward Tentacles'),
     User(id=None, name='ehkrabs', fullname='Eugene H. Krabs')]
)
```

В приведенном выше представлении используется коллекция с именем **IdentitySet**, которая по сути является набором Python, который во всех случаях хэширует идентификатор объекта (т. е. с использованием встроенной функции Python **id()**, а не функции Python **hash()**).

### Сброс данных (flushing)

Сессия [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) использует шаблон, известный как единица работы ([unit of work](https://docs.sqlalchemy.org/en/14/glossary.html#term-unit-of-work)). Обычно это означает, что он накапливает изменения по одному, но фактически не передает их в базу данных до тех пор, пока они не потребуются. Это позволяет ему принимать лучшие решения о том, как SQL **DML** (Data Manipulation Language - язык управления (манипулирования) данными) должен создаваться в транзакции на основе заданного набора ожидающих изменений. Когда он отправляет SQL в базу данных для отправки текущего набора изменений, этот процесс называется **flush**.

Мы можем проиллюстрировать процесс сброса вручную, вызвав метод [Session.flush()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.flush):

```python
>>> session.flush()
```

```sql
BEGIN (implicit)
INSERT INTO user_account (name, fullname) VALUES (?, ?)
[...] ('squidward', 'Squidward Tentacles')
INSERT INTO user_account (name, fullname) VALUES (?, ?)
[...] ('ehkrabs', 'Eugene H. Krabs')
```

Выше мы видим, что сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) был сначала вызван для отправки SQL, поэтому он создал новую транзакцию и выдал соответствующие операторы **INSERT** для двух объектов. Теперь **транзакция остается открытой** до тех пор, пока мы не вызовем любой из методов [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit), [Session.rollback()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.rollback) или [Session.close()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.close) объекта [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session).

Хотя [Session.flush()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.flush) можно использовать для ручной отправки ожидающих изменений в текущую транзакцию, обычно в этом нет необходимости, поскольку сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) имеет поведение, известное как **автозаполнение (autoflush)**, которое мы проиллюстрируем позже. Он также сбрасывает изменения всякий раз, когда вызывается [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit).

### Автоматически сгенерированные атрибуты первичного ключа

Как строки только вставлены, два созданных нами объекта Python находятся в состоянии, известном как постоянное ([persistent](https://docs.sqlalchemy.org/en/14/glossary.html#term-persistent)), когда они связаны с объектом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), в котором они были добавлены или загружены, и имеют множество других вариантов поведения, которые будут рассмотрены позже.

Другим следствием произошедшей операции **INSERT** было то, что ORM извлекал новые идентификаторы первичного ключа для каждого нового объекта; внутри обычно используется тот же метод доступа [CursorResult.inserted\_primary\_key](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.inserted\_primary\_key), который мы представили ранее. Объекты **squidward** и **krabs** теперь имеют эти новые идентификаторы первичного ключа, связанные с ними, и мы можем просмотреть их, обратившись к атрибуту **id**:

```python
>>> squidward.id
4
>>> krabs.id
5
```

{% hint style="info" %}
Совет:

Почему ORM выдает два отдельных оператора **INSERT**, когда можно было бы использовать [executemany](rabota-s-tranzakciyami-i-dbapi.md#otpravka-neskolkikh-parametrov)? Как мы увидим в следующем разделе, сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) при сбросе объектов всегда должен знать первичный ключ вновь вставленных объектов. Если используется такая функция, как автоинкремент **SQLite** (другие примеры включают **PostgreSQL** **IDENTITY** или **SERIAL**, использование последовательностей и т. д.), функция [CursorResult.inserted\_primary\_key](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult.inserted\_primary\_key) обычно требует, чтобы каждый **INSERT** выдавался по одной строке за раз. Если бы мы заранее предоставили значения для первичных ключей, ORM смогла бы лучше оптимизировать операцию. Некоторые серверные части базы данных, такие как [psycopg2](https://docs.sqlalchemy.org/en/14/dialects/postgresql.html#postgresql-psycopg2), также могут **INSERT** несколько строк одновременно, сохраняя при этом возможность извлекать значения первичного ключа.
{% endhint %}

### Карта идентичности (identity map)

Идентификатор первичного ключа объектов важен для сеанса [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), поскольку объекты теперь связаны с этим идентификатором в памяти с помощью функции, известной как карта идентификаторов ([identity map](https://docs.sqlalchemy.org/en/14/glossary.html#term-identity-map)). Карта идентификаторов — это хранилище в памяти, которое связывает все объекты, загруженные в данный момент в память, с идентификатором их первичного ключа. Мы можем наблюдать это, извлекая один из вышеуказанных объектов с помощью метода [Session.get()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.get), который вернет запись _**из карты идентификаторов, если она присутствует локально**_, в противном случае выдаст **SELECT**:

```python
>>> some_squidward = session.get(User, 4)
>>> some_squidward
User(id=4, name='squidward', fullname='Squidward Tentacles')
```

Важно отметить, что карта идентификаторов поддерживает **уникальный экземпляр** определенного объекта Python для определенного идентификатора базы данных в рамках определенного объекта [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session). Мы можем заметить, что **some\_squidward** ссылается на тот же объект, что и **squidward** ранее:

```python
>>> some_squidward is squidward
True
```

Карта идентичности — это важная функция, позволяющая манипулировать сложными наборами объектов в рамках транзакции без рассинхронизации.

### Фиксация (Commiting)

Можно еще многое сказать о том, как работает [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), что будет обсуждаться далее. На данный момент мы зафиксируем транзакцию, чтобы мы могли накопить знания о том, как делать **SELECT** строки, прежде чем исследовать больше поведения и функций ORM:

## Обновление ORM-объектов

В предыдущем разделе [Обновление и удаление строк с помощью Core](rabota-s-dannymi/obnovlenie-i-udalenie-strok-s-pomoshyu-core.md) мы представили конструкцию [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update), которая представляет инструкцию SQL **UPDATE**. При использовании ORM существует два способа использования этой конструкции. Основной способ заключается в том, что он генерируется автоматически как часть рабочего процесса ([unit of work](https://docs.sqlalchemy.org/en/14/glossary.html#term-unit-of-work)), используемого сеансом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), где инструкция **UPDATE** генерируется для каждого первичного ключа, соответствующего отдельным объектам, в которых есть изменения. Вторая форма **UPDATE** называется «**UPDATE** с поддержкой ORM» и позволяет нам явно использовать конструкцию [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update) с сеансом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session); это описано в следующем разделе.

Предположим, мы загрузили объект **User** для имени пользователя **sandy** в транзакцию (также демонстрируя метод [Select.filter\_by()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select.filter\_by) и метод [Result.scalar\_one()](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result.scalar\_one)):

```python
>>> sandy = session.execute(select(User).filter_by(name="sandy")).scalar_one()
```

```sql
BEGIN (implicit)
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = ?
[...] ('sandy',)
```

Объект Python **sandy**, как упоминалось ранее, действует как **прокси** для строки в базе данных, точнее для строки базы данных **с точки зрения текущей транзакции**, которая имеет идентификатор первичного ключа **2**:

```python
>>> sandy
User(id=2, name='sandy', fullname='Sandy Cheeks')
```

Если мы изменим атрибуты этого объекта, [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) отследит это изменение:

```python
>>> sandy.fullname = "Sandy Squirrel"
```

Объект появляется в коллекции под названием [Session.dirty](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.dirty), что указывает на то, что объект «грязный» (_dirty_):

```python
>>> sandy in session.dirty
True
```

Когда [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) в следующий раз выдаст сброс, будет выдано **UPDATE**, которое обновит это значение в базе данных. Как упоминалось ранее, сброс происходит автоматически до того, как мы выполним какой-либо **SELECT**, используя поведение, известное как **автосброс (autoflush)**. Мы можем напрямую запросить столбец **User.fullname** из этой строки, и мы вернем наше обновленное значение:

```python
>>> sandy_fullname = session.execute(
...     select(User.fullname).where(User.id == 2)
... ).scalar_one()
```

```sql
UPDATE user_account SET fullname=? WHERE user_account.id = ?
[...] ('Sandy Squirrel', 2)
SELECT user_account.fullname
FROM user_account
WHERE user_account.id = ?
[...] (2,)
```

```python
>>> print(sandy_fullname)
Sandy Squirrel
```

Выше мы видим, что мы запросили, чтобы сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) выполнил один оператор [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select). Однако испущенный SQL показывает, что **UPDATE** также был выпущен, что было процессом сброса (_flush_), выталкивающим ожидающие изменения. Объект **sandy** Python больше не считается грязным (_dirty_):

```python
>>> sandy in session.dirty
False
```

Однако обратите внимание, что мы **все еще находимся в транзакции**, и наши изменения не были отправлены в постоянное хранилище базы данных. Поскольку фамилия Сэнди на самом деле «Cheeks», а не «Squirell», мы исправим эту ошибку позже при откате транзакции. Но сначала мы внесем еще несколько изменений в данные.

{% hint style="info" %}
Смотри также:

[Flushing](https://docs.sqlalchemy.org/en/14/orm/session\_basics.html#session-flushing) — подробные сведения о процессе очистки, а также информация о настройке [Session.autoflush](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.params.autoflush).
{% endhint %}

### Операторы UPDATE с поддержкой ORM

Как упоминалось ранее, существует второй способ выдачи операторов **UPDATE** с точки зрения ORM, который известен как **оператор UPDATE с поддержкой ORM**. Это позволяет использовать универсальную инструкцию SQL **UPDATE**, которая может воздействовать на несколько строк одновременно. Например, чтобы создать **UPDATE**, которое изменит столбец **User.fullname** на основе значения в столбце **User.name**:

```python
>>> session.execute(
...     update(User).
...     where(User.name == "sandy").
...     values(fullname="Sandy Squirrel Extraordinaire")
... )
```

```sql
UPDATE user_account SET fullname=? WHERE user_account.name = ?
[...] ('Sandy Squirrel Extraordinaire', 'sandy')
```

```bash
<sqlalchemy.engine.cursor.CursorResult object ...>
```

При вызове оператора **UPDATE** с поддержкой ORM используется специальная логика для _**поиска объектов в текущем сеансе**_, которые соответствуют заданным критериям, чтобы они _**обновлялись новыми данными**_. Выше идентификатор объекта **sandy** был расположен в памяти и обновлен:

```python
>>> sandy.fullname
'Sandy Squirrel Extraordinaire'
```

Логика обновления называется параметром **synchronize\_session** и подробно описана в разделе UPDATE и DELETE с произвольным предложением WHERE ([UPDATE and DELETE with arbitrary WHERE clause](https://docs.sqlalchemy.org/en/14/orm/session\_basics.html#orm-expression-update-delete)).

{% hint style="info" %}
Смотри также:

[UPDATE and DELETE with arbitrary WHERE clause](https://docs.sqlalchemy.org/en/14/orm/session\_basics.html#orm-expression-update-delete) - описывает использование ORM функций [update()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.update) и [delete()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.delete), а также параметры синхронизации ORM.
{% endhint %}

## Удаление ORM-объектов

Чтобы завершить основные операции сохранения, отдельный объект ORM может быть помечен для удаления с помощью метода [Session.delete()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.delete). Давайте загрузим **patrick** из базы данных:

```python
>>> patrick = session.get(User, 3)
```

```sql
SELECT user_account.id AS user_account_id, user_account.name AS user_account_name,
user_account.fullname AS user_account_fullname
FROM user_account
WHERE user_account.id = ?
[...] (3,)
```

Если мы пометим **patrick** для удаления, как в случае с другими операциями, на самом деле ничего не произойдет, пока не произойдет сброс:

```python
>>> session.delete(patrick)
```

Текущее поведение ORM заключается в том, что **patrick** остается в сеансе [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) до тех пор, пока не продолжится сброс, что, как упоминалось ранее, происходит, если мы выдаем запрос:

```python
>>> session.execute(select(User).where(User.name == "patrick")).first()
```

```sql
SELECT address.id AS address_id, address.email_address AS address_email_address,
address.user_id AS address_user_id
FROM address
WHERE ? = address.user_id
[...] (3,)
DELETE FROM user_account WHERE user_account.id = ?
[...] (3,)
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = ?
[...] ('patrick',)
```

Выше **SELECT**, который мы просили выполнить, предварялся **DELETE**, что указывало на то, что отложенное удаление для **patrick** продолжается. Также был **SELECT** для таблицы адресов **address**, который был вызван ORM, ищущим строки в этой таблице, которые могут быть связаны с целевой строкой; это поведение является частью поведения, известного как каскад ([cascade](https://docs.sqlalchemy.org/en/14/glossary.html#term-cascade)), и может быть адаптировано для более эффективной работы, позволяя базе данных автоматически обрабатывать связанные строки в адресе **address**; в разделе удаления ([delete](https://docs.sqlalchemy.org/en/14/orm/cascades.html#cascade-delete)) есть все подробности об этом.

{% hint style="info" %}
Смотри также:

[delete](https://docs.sqlalchemy.org/en/14/orm/cascades.html#cascade-delete) — описывает, как настроить поведение [Session.delete()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.delete) с точки зрения того, как должны обрабатываться связанные строки в других таблицах.
{% endhint %}

Кроме того, экземпляр объекта **patrick**, который сейчас удаляется, больше не считается постоянным в сеансе [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), как показывает проверка содержания:

```python
>>> patrick in session
False
```

Однако, как и **UPDATE**, которые мы сделали для объекта **sandy**, каждое изменение, которое мы сделали здесь, является локальным для текущей транзакции, которая не станет постоянной, если мы ее не зафиксируем. Поскольку откат транзакции на данный момент более интересен, мы сделаем это в следующем разделе.

### Операторы DELETE с поддержкой ORM

Подобно операциям **UPDATE**, существует также версия **DELETE** с поддержкой ORM, которую мы можем проиллюстрировать, используя конструкцию [delete()](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.delete) с [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute). Он также имеет функцию, с помощью которой объекты с **не истекшим** сроком действия (см. просроченные - [expired](https://docs.sqlalchemy.org/en/14/glossary.html#term-expired)), которые соответствуют заданным критериям удаления, будут автоматически помечены как «удаленные» ([deleted](https://docs.sqlalchemy.org/en/14/glossary.html#term-deleted)) в сеансе [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session):

```python
>>> # обновить целевой объект только для демонстрационных целей,
>>> # не требуется для DELETE

>>> squidward = session.get(User, 4)
```

```sql
SELECT user_account.id AS user_account_id, user_account.name AS user_account_name,
user_account.fullname AS user_account_fullname
FROM user_account
WHERE user_account.id = ?
[...] (4,)
```

```python
>>> session.execute(delete(User).where(User.name == "squidward"))
```

```sql
DELETE FROM user_account WHERE user_account.name = ?
[...] ('squidward',)
```

```bash
<sqlalchemy.engine.cursor.CursorResult object at 0x...>
```

Личность **squidward**, как и личность **patrick**, теперь также находится в удаленном состоянии. Обратите внимание, что нам пришлось перезагрузить **squidward** выше, чтобы продемонстрировать это; если срок действия объекта истек, операция **DELETE** не будет тратить время на обновление просроченных объектов только для того, чтобы увидеть, что они были удалены:

```python
>>> squidward in session
False
```

## Откат назад (rolling back)

[Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) имеет метод [Session.rollback()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.rollback), который, как и ожидалось, выдает **ROLLBACK** для текущего SQL-соединения. Однако это также влияет на объекты, которые в настоящее время связаны с сеансом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), в нашем предыдущем примере это объект Python **sandy**. Хотя мы изменили полное имя объекта **sandy** на «Sandy Squirrel», мы хотим отменить это изменение. Вызов [Session.rollback()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.rollback) не только откатывает транзакцию, но и **закончит срок действия всех объектов**, в настоящее время связанных с этим сеансом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), что приведет к тому, что они будут обновляться при следующем доступе с использованием процесса, известного как ленивая загрузка ([lazy loading](https://docs.sqlalchemy.org/en/14/glossary.html#term-lazy-loading)):

```python
>>> session.rollback()
ROLLBACK
```

Чтобы рассмотреть процесс «истечения срока действия» более подробно, мы можем заметить, что у объекта Python **sandy** не осталось состояния в его Python **\_\_dict\_\_**, за исключением специального объекта внутреннего состояния SQLAlchemy:

```python
>>> sandy.__dict__
{'_sa_instance_state': <sqlalchemy.orm.state.InstanceState object at 0x...>}
```

Это «истекшее» состояние ([expired](https://docs.sqlalchemy.org/en/14/glossary.html#term-expired)); повторный доступ к атрибуту автоматически запустит новую транзакцию и обновит **sandy** с текущей строкой базы данных:

```python
>>> sandy.fullname
```

```sql
BEGIN (implicit)
SELECT user_account.id AS user_account_id, user_account.name AS user_account_name,
user_account.fullname AS user_account_fullname
FROM user_account
WHERE user_account.id = ?
[...] (2,)
```

```bash
'Sandy Cheeks'
```

Теперь мы можем заметить, что полная строка базы данных также была заполнена в **\_\_dict\_\_** объекта **sandy**:

```python
>>> sandy.__dict__  
{'_sa_instance_state': <sqlalchemy.orm.state.InstanceState object at 0x...>,
 'id': 2, 'name': 'sandy', 'fullname': 'Sandy Cheeks'}
```

Для удаленных объектов, когда мы ранее отметили, что **patrick** больше нет в сеансе, также восстанавливается идентификация этого объекта:

```python
>>> patrick in session
True
```

и, конечно, данные базы данных также присутствуют снова:

```python
>>> session.execute(select(User).where(User.name == 'patrick')).scalar_one() is patrick
True
```

```sql
SELECT user_account.id, user_account.name, user_account.fullname
FROM user_account
WHERE user_account.name = ?
[...] ('patrick',)
```

## Закрытие сеанса Session

В приведенных выше разделах мы использовали объект [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) вне менеджера контекста Python, то есть мы не использовали оператор **with**. Это нормально, однако, если мы делаем что-то таким образом, лучше всего явно закрыть сессию [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), когда мы закончим с ней:

```python
>>> session.close()
```

```sql
ROLLBACK
```

Закрытие сеанса [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), которое происходит, когда мы используем его в диспетчере контекста, выполняет следующие действия:

* Он высвобождает ([releases](https://docs.sqlalchemy.org/en/14/glossary.html#term-releases)) все ресурсы соединения в пул соединений, отменяя (например, откатывая) все транзакции, которые выполнялись. Это означает, что когда мы используем сеанс для выполнения некоторых задач только для чтения, а затем закрываем его, нам не нужно явно вызывать [Session.rollback()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.rollback), чтобы гарантировать откат транзакции; пул соединений обрабатывает это.
* Он **удаляет** все объекты из сеанса [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session). Это означает, что все объекты Python, которые мы загрузили для этого сеанса [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), такие как **sandy**, **patrick** и **squidward**, теперь находятся в состоянии, известном как отсоединенное ([detached](https://docs.sqlalchemy.org/en/14/glossary.html#term-detached)). В частности, отметим, что объекты, которые все еще находились в просроченном состоянии ([expired](https://docs.sqlalchemy.org/en/14/glossary.html#term-expired)), например, из-за вызова [Session.commit()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.commit), теперь нефункциональны, так как не содержат состояния текущей строки и больше не связаны с любой транзакцией базы данных, в которой необходимо обновить:

```python
>>> squidward.name
Traceback (most recent call last):
  ...
sqlalchemy.orm.exc.DetachedInstanceError: Instance <User at 0x...>
is not bound to a Session; attribute refresh operation cannot proceed
```

Отсоединенные объекты могут быть повторно связаны с тем же самым или новым сеансом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) с использованием метода [Session.add()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.add), который восстановит их связь с их конкретной строкой базы данных:

```python
>>> session.add(squidward)
>>> squidward.name
```

```sql
BEGIN (implicit)
SELECT user_account.id AS user_account_id, user_account.name AS user_account_name, user_account.fullname AS user_account_fullname
FROM user_account
WHERE user_account.id = ?
[...] (4,)
```

```bash
'squidward'
```

{% hint style="info" %}
Совет:

По возможности старайтесь избегать использования объектов в их отсоединенном состоянии. Когда сессия [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) закрыта, также очистите ссылки на все ранее прикрепленные объекты. В случаях, когда необходимы отсоединенные объекты, как правило, для немедленного отображения только что зафиксированных объектов для веб-приложения, когда сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) закрывается до отображения представления, установите для флага [Session.expire\_on\_commit](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.params.expire\_on\_commit) значение `False`.
{% endhint %}

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел руководства: [Работа со связанными объектами (relationship)](rabota-so-svyazannymi-obektami.md)
{% endhint %}
