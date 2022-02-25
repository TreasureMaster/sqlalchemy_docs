# func mapper()

## _function_ sqlalchemy.orm.mapper(_class\__, _local\_table=None_, _properties=None_, _primary\_key=None_, _non\_primary=False_, _inherits=None_, _inherit\_condition=None_, _inherit\_foreign\_keys=None_, _always\_refresh=False_, _version\_id\_col=None_, _version\_id\_generator=None_, _polymorphic\_on=None_, _\_polymorphic\_map=None_, _polymorphic\_identity=None_, _concrete=False_, _with\_polymorphic=None_, _polymorphic\_load=None_, _allow\_partial\_pks=True_, _batch=True_, _column\_prefix=None_, _include\_properties=None_, _exclude\_properties=None_, _passive\_updates=True_, _passive\_deletes=False_, _confirm\_deleted\_rows=True_, _eager\_defaults=False_, _legacy\_is\_orphan=False_, _\_compiled\_cache\_size=100_)

Прямой конструктор для нового объекта Mapper.

Функция [mapper()](func-mapper.md#function-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary) обычно вызывается с использованием объекта реестра [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl) в [декларативном](../otobrazhenie-klassov-python.md#deklarativnoe-sopostavlenie-declarative-mapping) или [императивном](../otobrazhenie-klassov-python.md#imperativnoe-ono-zhe-klassicheskoe-sopostavlenie-imperative-mapping) стилях сопоставления.

{% hint style="warning" %}
**Изменено в версии 1.4**: Функция [mapper()](func-mapper.md#function-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary) не должна вызываться напрямую для классического отображения; для классической конфигурации сопоставления используйте метод [registry.map\_imperatively()](class-registry.md#method-sqlalchemy.orm.registry.map\_imperatively-class\_-local\_table-none-kw). Функция [mapper()](func-mapper.md#function-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary) может стать приватной в будущих версиях.
{% endhint %}

Параметры, задокументированные ниже, могут быть переданы либо методу [registry.map\_imperatively()](class-registry.md#method-sqlalchemy.orm.registry.map\_imperatively-class\_-local\_table-none-kw), либо могут быть переданы в декларативном атрибуте класса **\_\_mapper\_args\_\_**, описанном в разделе [Параметры конфигурации Mapper с декларативным](../otobrazhenie-klassov-v-deklarativnom-stile/konfiguraciya-sopostavleniya-v-deklarativnom-otobrazhenii.md#parametry-konfiguracii-mapper-s-declarative).

#### Параметры:

* **class\_** - Класс, который нужно сопоставить. При использовании **Declarative** этот аргумент автоматически передается как сам объявленный класс.
* **local\_table** - Таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) или другой _selectable_ объект, на который сопоставляется класс. Может быть `None`, если этот модуль отображения наследуется от другого модуля отображения с использованием наследования одной таблицы. При использовании **Declarative** этот аргумент автоматически передается расширением на основе того, что настроено с помощью аргумента __ **\_\_table\_\_** или с помощью таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), созданной в результате присутствия аргументов **\_\_tablename\_\_** и [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column).
* **always\_refresh** - Если задано значение `True`, все операции запроса для этого сопоставленного класса будут перезаписывать все данные в экземплярах объектов, которые уже существуют в рамках сеанса, стирая любые изменения в памяти любой информации, загруженной из базы данных. Использование этого флага крайне не рекомендуется; в качестве альтернативы см. метод [Query.populate\_existing()](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query.populate\_existing).
* **allow\_partial\_pks** - По умолчанию `True`. Указывает, что составной первичный ключ с некоторыми значениями **NULL** следует рассматривать как возможно существующий в базе данных. Это влияет на то, будет ли mapper назначать входящую строку существующему идентификатору, а также будет ли [Session.merge()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.merge) сначала проверять базу данных на наличие определенного значения первичного ключа. «Частичный первичный ключ» может возникнуть, например, если он сопоставлен с **OUTER JOIN**.
* **batch** - По умолчанию установлено значение `True`, что указывает на то, что операции сохранения нескольких сущностей могут быть объединены вместе для повышения эффективности. Значение `False` указывает, что экземпляр будет полностью сохранен перед сохранением следующего экземпляра. Это используется в крайне редких случаях, когда прослушиватель [MapperEvents](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.MapperEvents) требует вызова между операциями сохранения отдельных строк.
* **column\_prefix** - Строка, которая будет добавляться к имени сопоставленного атрибута, когда объекты [Colum](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column)n автоматически назначаются в качестве атрибутов сопоставленному классу. Не влияет на явно указанные свойства на основе столбца. См. пример в разделе [Именование всех столбцов с префиксом](../otobrazhenie-kolonok-i-vyrazhenii/otobrazhenie-kolonok-tablicy.md#imenovanie-vsekh-stolbcov-s-prefiksom).
* **concrete** - Если значение равно `True`, это указывает, что этот преобразователь должен использовать наследование конкретной таблицы со своим родительским преобразователем. См. пример в разделе [Наследование конкретных таблиц](../otobrazhenie-klassov-v-ierarkhii-nasledovaniya/nasledovanie-konkretnoi-tablicy.md#nasledovanie-konkretnoi-tablicy).
* **confirm\_deleted\_rows** - по умолчанию `True`; когда происходит **DELETE** еще одной строки на основе определенных первичных ключей, выдается предупреждение, когда количество совпадающих строк не равно ожидаемому количеству строк. Для этого параметра можно задать значение `False`, чтобы обрабатывать случай, когда правила базы данных **ON DELETE CASCADE** могут автоматически удалять некоторые из этих строк. Предупреждение может быть изменено на исключение в будущем выпуске.

{% hint style="warning" %}
**Новое в версии 0.9.4**: - добавлен **mapper.confirm\_deleted\_rows**, а также условная проверка совпадающих строк при удалении.
{% endhint %}

* **eager\_defaults** - если `True`, ORM будет немедленно извлекать значение сгенерированных сервером значений по умолчанию после **INSERT** или **UPDATE**, а не оставлять их с истекшим сроком действия для извлечения при следующем доступе. Это можно использовать для схем событий, когда сгенерированные сервером значения необходимы непосредственно перед завершением сброса. По умолчанию эта схема выдает отдельный оператор **SELECT** для каждой вставленной или обновленной строки, что может значительно увеличить производительность. Однако, если целевая база данных поддерживает [RETURNING](https://docs.sqlalchemy.org/en/14/glossary.html#term-RETURNING), значения по умолчанию будут возвращены вместе с инструкцией **INSERT** или **UPDATE**, что может значительно повысить производительность приложения, которому требуется частый доступ к только что сгенерированным значениям по умолчанию сервера.

{% hint style="info" %}
Смотри также:

[Fetching Server-Generated Defaults](https://docs.sqlalchemy.org/en/14/orm/persistence\_techniques.html#orm-server-defaults)
{% endhint %}

{% hint style="warning" %}
**Изменено в версии 0.9.0**: опция **eager\_defaults** теперь может использовать [RETURNING](https://docs.sqlalchemy.org/en/14/glossary.html#term-RETURNING) для бэкендов, которые его поддерживают.
{% endhint %}

* **exclude\_properties** - Список или набор имен строковых столбцов, которые необходимо исключить из сопоставления. Пример см. в разделе [Сопоставление подмножества столбцов таблицы](../otobrazhenie-kolonok-i-vyrazhenii/otobrazhenie-kolonok-tablicy.md#otobrazhenie-podmnozhestva-stolbcov-tablicy).
* **include\_properties** - Включающий список или набор строковых имен столбцов для сопоставления. Пример см. в разделе [Сопоставление подмножества столбцов таблицы](../otobrazhenie-kolonok-i-vyrazhenii/otobrazhenie-kolonok-tablicy.md#otobrazhenie-podmnozhestva-stolbcov-tablicy).
* **inherits** - Сопоставленный класс или соответствующий Mapper одного из них, указывающий суперкласс, от которого этот Mapper должен _**наследоваться**_. Сопоставляемый класс здесь должен быть подклассом другого класса сопоставителя mapper. При использовании **Declarative** этот аргумент передается автоматически в результате естественной иерархии объявленных классов.

{% hint style="info" %}
Смотри также:

[Mapping Class Inheritance Hierarchies](../otobrazhenie-klassov-v-ierarkhii-nasledovaniya/)
{% endhint %}

* **inherit\_condition** - Для наследования объединенных таблиц — SQL-выражение, определяющее способ соединения двух таблиц; по умолчанию используется естественное соединение между двумя таблицами.
* **inherit\_foreign\_keys** - Когда используется **inherit\_condition** и в присутствующих столбцах отсутствует конфигурация [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey), этот параметр можно использовать для указания того, какие столбцы являются «внешними». В большинстве случаев можно оставить `None`.
* **legacy\_is\_orphan** - Boolean, по умолчанию `False`. Если значение равно `True`, указывает, что к объектам, сопоставленным этим преобразователем, должно применяться «устаревшее» рассмотрение сиротства, что означает, что ожидающий (то есть не постоянный) объект автоматически удаляется из сеанса-владельца [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) только тогда, когда он деассоциируется со _**всеми**_ родителями, указывающие каскад **delete-orphan** по отношению к этому преобразователю. Новое поведение по умолчанию заключается в том, что объект автоматически удаляется, когда он деассоциируется с любым из его родителей, которые определяют каскад **delete-orphan**. Это поведение более совместимо с поведением постоянного объекта и позволяет вести себя согласованно в большем количестве сценариев независимо от того, был ли сброшен объект-сирота или нет. См. примечание об изменении и пример в разделе Рассмотрение «ожидающего» объекта как «сиротского» стало более агрессивным для получения более подробной информации об этом изменении ([The consideration of a “pending” object as an “orphan” has been made more aggressive](https://docs.sqlalchemy.org/en/14/changelog/migration\_08.html#legacy-is-orphan-addition)).
* **non\_primary** - Указывает, что этот mapper является дополнением к «основному» mapper, то есть тому, который используется для персистентности. Созданный здесь Mapper может использоваться для специального сопоставления класса с альтернативным выбираемым _**selectable**_, только для загрузки.

{% hint style="danger" %}
**Устарело, начиная с версии 1.3**: параметр **mapper.non\_primary** устарел и будет удален в будущем выпуске. Функциональность неосновных преобразователей теперь лучше подходит для использования конструкции [AliasedClass](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.util.AliasedClass), которую также можно использовать в качестве цели [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) в версии 1.3.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Relationship to Aliased Class](https://docs.sqlalchemy.org/en/14/orm/join\_conditions.html#relationship-aliased-class) - новый шаблон, который устраняет необходимость в флаге **Mapper.non\_primary**.
{% endhint %}

*   **passive\_deletes** - Указывает поведение **DELETE** для столбцов внешнего ключа при удалении объекта наследования объединенной таблицы. По умолчанию `False` для базового mapper; для наследующего преобразователя по умолчанию используется значение `False`, если только значение не установлено в `True` в преобразователе суперкласса.

    Когда значение равно `True`, предполагается, что **ON DELETE CASCADE** настроен для отношений внешнего ключа, которые связывают таблицу этого преобразователя с таблицей его суперкласса, поэтому, когда единица работы пытается удалить сущность, ей нужно только выдать инструкцию **DELETE** для суперкласса таблицы, а не эту таблицу.

    Когда `False`, инструкция **DELETE** генерируется для этой таблицы сопоставления отдельно. Если атрибуты первичного ключа, локальные для этой таблицы, выгружены, то для проверки этих атрибутов необходимо выполнить команду **SELECT**; обратите внимание, что столбцы первичного ключа подкласса объединенной таблицы не являются частью «первичного ключа» объекта в целом.

    Обратите внимание, что значение `True` **всегда** принудительно устанавливается в преобразователях подклассов; то есть для суперкласса невозможно указать **passive\_deletes** без того, чтобы это не вступило в силу для всех преобразователей подкласса.

{% hint style="warning" %}
**Новое в версии 1.1**.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Using foreign key ON DELETE cascade with ORM relationships](https://docs.sqlalchemy.org/en/14/orm/cascades.html#passive-deletes) - описание аналогичной функции, используемой с [`relationship()`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship)

**mapper.passive\_updates** - поддержка **ON UPDATE CASCADE** для mappers наследования объединенных таблиц
{% endhint %}

*   **passive\_updates** - Указывает поведение **UPDATE** для столбцов внешнего ключа, когда столбец первичного ключа изменяется в сопоставлении наследования объединенной таблицы. По умолчанию `True`.

    При значении `True` предполагается, что **ON UPDATE CASCADE** настроен для внешнего ключа в базе данных и что база данных будет обрабатывать распространение **UPDATE** из исходного столбца в зависимые столбцы в строках объединенной таблицы.

    При значении `False` предполагается, что база данных не обеспечивает ссылочную целостность и не будет запускать собственную операцию **CASCADE** для обновления. Единица рабочего процесса выдаст оператор **UPDATE** для зависимых столбцов во время изменения первичного ключа.

{% hint style="info" %}
Смотри также:

[Mutable Primary Keys / Update Cascades](https://docs.sqlalchemy.org/en/14/orm/relationship\_persistence.html#passive-updates) - описание аналогичной функции, используемой с [`relationship()`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship)

**mapper.passive\_deletes** - поддержка **ON DELETE CASCADE** для mappers наследования объединенных таблиц
{% endhint %}

* **polymorphic\_load** - Определяет поведение «полиморфной загрузки» для подкласса в иерархии наследования (только объединенное и однотабличное наследование). Допустимые значения:
  * **"inline"** - указывает, что этот класс должен быть частью **with\_polymorphic** mappers, например, его столбцы будут включены в запрос **SELECT** к базе.
  * **"selectin"** - указывает, что при загрузке экземпляров этого класса будет выдан дополнительный **SELECT** для извлечения столбцов, характерных для этого подкласса. **SELECT** использует **IN** для одновременной выборки нескольких подклассов.

{% hint style="warning" %}
**Новое в версии 1.2**.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Setting with\_polymorphic at mapper configuration time](https://docs.sqlalchemy.org/en/14/orm/inheritance\_loading.html#with-polymorphic-mapper-config)

[Polymorphic Selectin Loading](https://docs.sqlalchemy.org/en/14/orm/inheritance\_loading.html#polymorphic-selectin)
{% endhint %}

* **polymorphic\_on** - Указывает столбец, атрибут или выражение SQL, используемые для определения целевого класса для входящей строки, когда присутствуют наследуемые классы. Это значение обычно представляет собой объект [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), присутствующий в отображаемой таблице [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table):

```python
class Employee(Base):
    __tablename__ = 'employee'

    id = Column(Integer, primary_key=True)
    discriminator = Column(String(50))

    __mapper_args__ = {
        "polymorphic_on":discriminator,
        "polymorphic_identity":"employee"
    }
```

Его также можно указать как выражение SQL, как в этом примере, где мы используем конструкцию [case()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.case) для обеспечения условного подхода:

```python
class Employee(Base):
    __tablename__ = 'employee'

    id = Column(Integer, primary_key=True)
    discriminator = Column(String(50))

    __mapper_args__ = {
        "polymorphic_on":case([
            (discriminator == "EN", "engineer"),
            (discriminator == "MA", "manager"),
        ], else_="employee"),
        "polymorphic_identity":"employee"
    }
```

Он также может относиться к любому атрибуту, сконфигурированному с помощью [column\_property()](../otobrazhenie-kolonok-i-vyrazhenii/otobrazhenie-kolonok-tablicy.md#function-sqlalchemy.orm.column\_property-columns-kwargs), или к строковому имени одного из них:

```python
class Employee(Base):
    __tablename__ = 'employee'

    id = Column(Integer, primary_key=True)
    discriminator = Column(String(50))
    employee_type = column_property(
        case([
            (discriminator == "EN", "engineer"),
            (discriminator == "MA", "manager"),
        ], else_="employee")
    )

    __mapper_args__ = {
        "polymorphic_on":employee_type,
        "polymorphic_identity":"employee"
    }
```

При настройке **polymorphic\_on** для ссылки на атрибут или выражение, которых нет в локально сопоставленной таблице [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), но значение дискриминатора должно быть сохранено в базе данных, значение дискриминатора не устанавливается автоматически для новых экземпляров; это должен обрабатывать пользователь либо вручную, либо с помощью прослушивателей событий. Типичный подход к созданию такого слушателя выглядит следующим образом:

```python
from sqlalchemy import event
from sqlalchemy.orm import object_mapper

@event.listens_for(Employee, "init", propagate=True)
def set_identity(instance, *arg, **kw):
    mapper = object_mapper(instance)
    instance.discriminator = mapper.polymorphic_identity
```

Там, где выше, мы присваиваем значение **polymorphic\_identity** для сопоставленного класса атрибуту _**discriminator**_, таким образом сохраняя значение в столбце _**discriminator**_ в базе данных.

{% hint style="danger" %}
В настоящее время **может быть установлен только один столбец дискриминатора**, обычно в самом базовом классе в иерархии. «Каскадные» полиморфные столбцы пока не поддерживаются.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Mapping Class Inheritance Hierarchies](../otobrazhenie-klassov-v-ierarkhii-nasledovaniya/)
{% endhint %}

* **polymorphic\_identity** - Указывает значение, которое идентифицирует этот конкретный класс как возвращенное выражением столбца, на которое ссылается параметр **polymorphic\_on**. По мере получения строк значение, соответствующее выражению столбца **polymorphic\_on**, сравнивается с этим значением, указывая, какой подкласс следует использовать для вновь реконструированного объекта.
* **properties** - Словарь, сопоставляющий строковые имена атрибутов объекта с экземплярами [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty), которые определяют поведение сохраняемости этого атрибута. Обратите внимание, что объекты [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), присутствующие в сопоставленной таблице [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), автоматически помещаются в экземпляры **ColumnProperty** при сопоставлении, если они не переопределены. При использовании **Declarative** этот аргумент передается автоматически на основе всех экземпляров [MapperProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty), объявленных в объявленном теле класса.
* **primary\_key** - Список объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), которые определяют первичный ключ, который будет использоваться против единицы, выбираемой этим mapper. Обычно это просто первичный ключ **local\_table**, но здесь его можно переопределить.
* **version\_id\_col** - [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), который будет использоваться для хранения идентификатора текущей версии строк в таблице. Это используется для обнаружения одновременных обновлений или наличия устаревших данных во флеше. Методология заключается в том, чтобы определить, не соответствует ли оператор **UPDATE** последнему известному идентификатору версии, возникает исключение [StaleDataError](https://docs.sqlalchemy.org/en/14/orm/exceptions.html#sqlalchemy.orm.exc.StaleDataError). По умолчанию столбец должен иметь целочисленный тип [Integer](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Integer), если только **version\_id\_generator** не указывает альтернативный генератор версий.

{% hint style="info" %}
Смотри также:

[Configuring a Version Counter](../konfigurirovanie-schetchika-versii.md) - обсуждение подсчета версий и обоснования.
{% endhint %}

* **version\_id\_generator** - Определяет, как должны генерироваться новые идентификаторы версий. По умолчанию установлено значение `None`, что указывает на использование простой схемы подсчета целых чисел. Чтобы предоставить пользовательскую схему управления версиями, предоставьте вызываемую функцию в форме:

```python
def generate_version(version):
    return next_version
```

В качестве альтернативы можно использовать функции управления версиями на стороне сервера, такие как триггеры, или программные схемы управления версиями вне генератора идентификатора версии, указав значение `False`. См. [Счетчики версий на стороне сервера](../konfigurirovanie-schetchika-versii.md#schetchiki-versii-na-storone-servera) для обсуждения важных моментов при использовании этой опции.

{% hint style="warning" %}
**Новое в версии 0.9.0**: **version\_id\_generator** поддерживает генерацию номера версии на стороне сервера.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Custom Version Counters / Types](../konfigurirovanie-schetchika-versii.md#polzovatelskie-schetchiki-tipy-versii)

[Server Side Version Counters](../konfigurirovanie-schetchika-versii.md#schetchiki-versii-na-storone-servera)
{% endhint %}

* **with\_polymorphic** - Кортеж в форме `(<classes>, <selectable>)`, указывающий стиль «полиморфной» загрузки по умолчанию, то есть какие таблицы запрашиваются одновременно. `<classes>` — это любой одиночный или список mappers и/или классов, указывающий унаследованные классы, которые должны быть загружены сразу. Специальное значение `'*'` может использоваться для указания того, что все нисходящие классы должны быть загружены немедленно. Второй аргумент кортежа `<selectable>` указывает выбираемый объект, который будет использоваться для запроса нескольких классов.

{% hint style="info" %}
Смотри также:

[Using with\_polymorphic](https://docs.sqlalchemy.org/en/14/orm/inheritance\_loading.html#with-polymorphic) - обсуждение методов полиморфных запросов.
{% endhint %}
