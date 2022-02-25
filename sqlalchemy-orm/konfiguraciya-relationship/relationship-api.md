# Relationship API

| Название объекта                                                                                                                                                  | Описание                                                                                                                                             |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| [backref](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.backref)(name, \*\*kwargs)                                                  | Создает обратную ссылку с явными аргументами ключевого слова, которые являются теми же самыми аргументами, которые можно отправить в relationship(). |
| [dynamic\_loader](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.dynamic\_loader)(argument, \*\*kw)                                  | Создает динамически загружаемое свойство сопоставления.                                                                                              |
| [foreign](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.foreign)(expr)                                                              | Аннотирует часть выражения первичного соединения «внешней» аннотацией.                                                                               |
| [relation](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relation)(\*arg, \*\*kw)                                                   | Синоним отношения relationship().                                                                                                                    |
| [relationship](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship)(argument\[, secondary, primaryjoin, secondaryjoin, ...]) | Обеспечивает связь между двумя сопоставленными классами.                                                                                             |
| [remote](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.remote)(expr)                                                                | Аннотирует часть выражения первичного соединения (primaryjoin) «удаленной» аннотацией.                                                               |

### _function_ sqlalchemy.orm.relationship(_argument_, _secondary=None_, _primaryjoin=None_, _secondaryjoin=None_, _foreign\_keys=None_, _uselist=None_, _order\_by=False_, _backref=None_, _back\_populates=None_, _overlaps=None_, _post\_update=False_, _cascade=False_, _viewonly=False_, _lazy='select'_, _collection\_class=None_, _passive\_deletes=False_, _passive\_updates=True_, _remote\_side=None_, _enable\_typechecks=True_, _join\_depth=None_, _comparator\_factory=None_, _single\_parent=False_, _innerjoin=False_, _distinct\_target\_key=None_, _doc=None_, _active\_history=False_, _cascade\_backrefs=True_, _load\_on\_pending=False_, _bake\_queries=True_, _\_local\_remote\_pairs=None_, _query\_class=None_, _info=None_, _omit\_join=None_, _sync\_backref=None_, _\_legacy\_inactive\_history\_style=False_)

Обеспечивает связь между двумя сопоставленными классами.

Это соответствует отношениям родитель-потомок или ассоциативной таблице. Сконструированный класс является экземпляром [RelationshipProperty](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty).

Типичное отношение [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), используемое в классическом отображении:

```python
mapper(Parent, properties={
  'children': relationship(Child)
})
```

Некоторые аргументы, принимаемые функцией [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), необязательно принимают вызываемую функцию, которая при вызове возвращает желаемое значение. Вызываемый объект вызывается родительским преобразователем [Mapper](../konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal) во время «инициализации преобразователя», что происходит только при первом использовании преобразователей и предполагается, что это происходит после создания всех сопоставлений. Это можно использовать для решения проблем порядка объявления и других проблем с зависимостями, например, если дочерний элемент **Child** объявлен _**ниже**_ родительского **Parent** в том же файле:

```python
mapper(Parent, properties={
    "children":relationship(lambda: Child,
                        order_by=lambda: Child.id)
})
```

При использовании расширения декларативных расширений ([Declarative Extensions](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/index.html)) декларативный инициализатор позволяет передавать строковые аргументы в отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for). Эти строковые аргументы преобразуются в вызываемые объекты, которые оценивают строку как код Python, используя декларативный реестр классов в качестве пространства имен. Это позволяет выполнять автоматический поиск связанных классов по их строковым именам и устраняет необходимость импорта связанных классов в локальное пространство модулей до того, как будут объявлены зависимые классы. По-прежнему требуется, чтобы модули, в которых появляются эти связанные классы, были импортированы в любом месте приложения в какой-то момент до фактического использования связанных сопоставлений, в противном случае будет вызвана ошибка поиска, когда отношение [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) попытается разрешить ссылку строки на родственный класс. Пример класса с разрешением строк выглядит следующим образом:

```python
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    children = relationship("Child", order_by="Child.id")
```

{% hint style="info" %}
Смотри также:

[Relationship Configuration](./) - Полная вводная и справочная документация по [`relationship()`](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for).

[Building a Relationship](../uchebnoe-posobie-po-relyacionnym-obektam-1.x-api/tutorial-chast-3-relationship-join.md#postroenie-svyazei-relationship) - Введение в учебник ORM.
{% endhint %}

#### Параметры:

* **argument** - Сопоставленный класс или фактический экземпляр [Mapper](../konfiguraciya-mapper/api-klassa-otobrazheniya-mapping-class/class-mapper.md#class-sqlalchemy.orm.mapper-class\_-local\_table-none-properties-none-primary\_key-none-non\_primary-fal), представляющий цель отношения. **relationship.argument** также может быть передан как _**вызываемая функция**_, которая оценивается во время инициализации преобразователя, и может быть передана как _**строковое имя**_ при использовании **Declarative**.

{% hint style="danger" %}
До версии SQLAlchemy 1.3.16 это значение интерпретировалось с помощью функции Python _**eval()**_. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ**. Подробную информацию о декларативной оценке аргументов отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

{% hint style="info" %}
Смотри также:

[Configuring Relationships](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-configuring-relationships) - дополнительные сведения о конфигурации отношений при использовании Declarative.
{% endhint %}

* **secondary** - Для отношения «многие ко многим» указывает промежуточную таблицу и обычно является экземпляром [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table). В менее распространенных случаях аргумент также может быть указан как конструкция псевдонима [Alias](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Alias) или даже как конструкция соединения [Join](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Join). Отношение **relationship.secondary** также может быть передано как вызываемая функция, которая оценивается во время инициализации преобразователя. При использовании **Declarative** это также может быть строковым аргументом, указывающим имя таблицы [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), которая присутствует в коллекции метаданных [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData), связанной с таблицей [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table), отображаемой родителем.

{% hint style="danger" %}
При передаче в виде строки, которую может вычислить Python, аргумент интерпретируется с помощью функции Python _**eval()**_. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ**. Подробную информацию о декларативной оценке аргументов отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

Аргумент ключевого слова **relationship.secondary** обычно применяется в случае, когда промежуточная таблица Table иначе не выражается в каком-либо прямом сопоставлении классов. Если «вторичная» таблица также явно отображается в другом месте (например, как в [объекте ассоциации](bazovye-shablony-relationship.md#associativnyi-obekt)), следует рассмотреть возможность применения флага **relationship.viewonly**, чтобы это отношение [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) не использовалось для операций постоянства, которые могут конфликтовать с операциями шаблона объекта ассоциации.

{% hint style="info" %}
Смотри также:

[Many To Many](bazovye-shablony-relationship.md#mnogie-ko-mnogim-many-to-many) - Справочный пример «многие ко многим».

[Building a Many To Many Relationship](../uchebnoe-posobie-po-relyacionnym-obektam-1.x-api/tutorial-chast-4-eager-load-delete.md#postroenie-otnosheniya-mnogie-ko-mnogim) - Учебник ORM: введение в отношения «многие ко многим».

[Self-Referential Many-to-Many Relationship](nastroika-soedineniya-join-otnoshenii.md#samoreferentnaya-svyaz-mnogie-ko-mnogim) - Особенности использования многие-ко-многим в случае самореференции.

[Configuring Many-to-Many Relationships](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-many-to-many) - Дополнительные возможности при использовании Declarative.

[Association Object](bazovye-shablony-relationship.md#associativnyi-obekt) - альтернатива [`relationship.secondary`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.secondary) при составлении отношений таблицы ассоциаций, позволяя указывать дополнительные атрибуты в таблице ассоциаций.

[Composite “Secondary” Joins](nastroika-soedineniya-join-otnoshenii.md#kompozitnye-vtorichnye-soedineniya) - менее используемый шаблон, который в некоторых случаях может позволить использовать сложные условия SQL [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for).
{% endhint %}

{% hint style="warning" %}
**Новое в версии 0.9.2**: отношение **relationship.secondary** работает более эффективно при обращении к экземпляру [Join](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Join).
{% endhint %}

* **active\_history=False** - Значение `True` указывает, что «предыдущее» значение для ссылки «многие к одному» должно быть загружено при замене, если оно еще не загружено. Обычно логике отслеживания истории для простых операций «многие к одному» необходимо знать только «новое» значение, чтобы выполнить сброс. Этот флаг доступен для приложений, которые используют [get\_history()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.attributes.get\_history), которым также необходимо знать «предыдущее» значение атрибута.
* **backref** - Указывает строковое имя свойства, которое должно быть помещено в связанный класс преобразователя, который будет обрабатывать это отношение в другом направлении. Другое свойство будет создано автоматически при настройке mappers. Также может передаваться как объект backref() для управления конфигурацией нового отношения.

{% hint style="info" %}
Смотри также:

[Linking Relationships with Backref](svyazyvanie-otnoshenii-s-backref.md#svyazyvanie-otnoshenii-s-backref) - Вводная документация и примеры.

[`relationship.back_populates`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.back\_populates) - альтернативная форма спецификации обратной ссылки.

[`backref()`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.backref) - позволяет контролировать конфигурацию отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) при использовании [`relationship.backref`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref).
{% endhint %}

* **back\_populates** - Принимает строковое имя и имеет то же значение, что и **relationship.backref**, за исключением того, что дополняющее свойство **не создается** автоматически, а вместо этого должно быть явно настроено в другом преобразователе. Дополняющее свойство также должно указывать на **relationship.back\_populates** для этого отношения, чтобы обеспечить правильное функционирование.

{% hint style="info" %}
Смотри также:

[Linking Relationships with Backref](svyazyvanie-otnoshenii-s-backref.md#svyazyvanie-otnoshenii-s-backref) - Вводная документация и примеры.

[`relationship.backref`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.backref) - альтернативная форма спецификации обратной ссылки.
{% endhint %}

* **overlaps** - Строковое имя или разделенный запятыми набор имен других отношений в этом преобразователе, дочернем преобразователе или целевом преобразователе, с которым это отношение может записывать те же внешние ключи при сохранении. Единственный эффект, который это имеет, заключается в устранении предупреждения о том, что это отношение будет конфликтовать с другим при сохранении. Это используется для таких отношений, которые действительно могут конфликтовать друг с другом при записи, но приложение гарантирует, что такие конфликты не возникнут.

{% hint style="warning" %}
**Новое в версии 1.4**.
{% endhint %}

{% hint style="info" %}
Отношение X скопирует столбец Q в столбец P, что конфликтует с отношениями: «Y» — пример использования ([relationship X will copy column Q to column P, which conflicts with relationship(s): ‘Y’](https://docs.sqlalchemy.org/en/14/errors.html#error-qzyx))
{% endhint %}

* **bake\_queries=True** - Включите [лямбда-кэширование](https://docs.sqlalchemy.org/en/14/core/connections.html#engine-lambda-caching) для стратегий загрузчика, если это применимо, что повышает производительность построения конструкций SQL, используемых стратегиями загрузчика, в дополнение к обычному кэшированию операторов SQL, используемому в SQLAlchemy. В настоящее время этот параметр применяется только к стратегиям загрузчика «**lazy**» и «**selectin**». Как правило, нет причин устанавливать для этого параметра значение `False`.

{% hint style="warning" %}
**Изменено в версии 1.4**: загрузчики отношений больше не используют предыдущую систему «backed query» кэширования запросов. «lazy» и «selectin» загрузчики используют систему «лямбда-кеша» для создания конструкций SQL, а также обычную систему кэширования SQL, которая используется в SQLAlchemy начиная с серии 1.4.
{% endhint %}

* **cascade** - Разделенный запятыми список каскадных правил, который определяет, как операции сеанса должны «каскадироваться» от родительского объекта к дочернему. По умолчанию это значение `False`, что означает, что следует использовать каскад по умолчанию — этот каскад по умолчанию — **«save-update, merge»**. Доступные каскады: **save-update**, **merge**, **expunge**, **delete**, **delete-orphan** и **refresh-expire**. Дополнительный параметр **all** указывает на сокращение для **«save-update, merge, refresh-expire, expunge, delete»** и часто используется как **«all, delete-orphan»**, чтобы указать, что связанные объекты должны следовать вместе с родительским объектом во всех случаях и удаляться при деассоциации.

{% hint style="info" %}
Смотри также:

[Cascades](https://docs.sqlalchemy.org/en/14/orm/cascades.html#unitofwork-cascades) - Полная информация о каждом из доступных вариантов каскадирования.

[Configuring delete/delete-orphan Cascade](../uchebnoe-posobie-po-relyacionnym-obektam-1.x-api/tutorial-chast-4-eager-load-delete.md#nastroika-kaskadnogo-udaleniya-udaleniya-zombi) - Учебный пример, описывающий каскад удаления.
{% endhint %}

* **cascade\_backrefs=True** - Логическое значение, указывающее, должен ли каскад **save-update** работать вместе с событием присваивания, перехваченным обратной ссылкой. Если установлено значение `False`, атрибут, управляемый этим отношением, не будет каскадировать входящий переходный объект в сеанс постоянного родителя, если событие получено через обратную ссылку.

{% hint style="warning" %}
**Устарело, начиная с версии 1.4**: флаг **relationship.cascade\_backrefs** будет по умолчанию иметь значение `False` во всех случаях в SQLAlchemy 2.0.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Controlling Cascade on Backrefs](https://docs.sqlalchemy.org/en/14/orm/cascades.html#backref-cascade) - Полное обсуждение и примеры того, как параметр [`relationship.cascade_backrefs`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.cascade\_backrefs) используется.
{% endhint %}

* **collection\_class** - Класс или вызываемый объект, который возвращает новый объект, содержащий список. Будет использоваться вместо простого списка для хранения элементов.

{% hint style="info" %}
Смотри также:

[Customizing Collection Access](konfiguraciya-kollekcii-i-metody-raboty-s-nimi/nastroika-dostupa-k-kollekcii.md) - Вводная документация и примеры.
{% endhint %}

* **comparator\_factory** - Класс, расширяющий [Comparator](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.RelationshipProperty.Comparator), который обеспечивает генерацию пользовательского предложения SQL для операций сравнения.

{% hint style="info" %}
Смотри также:

[`PropComparator`](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.PropComparator) - некоторые подробности о переопределении компараторов на этом уровне.

[Operator Customization](../konfiguraciya-mapper/otobrazhenie-kolonok-i-vyrazhenii/izmenenie-povedeniya-atributov.md#individualnaya-nastroika-operatora) - Краткое введение в эту функцию.
{% endhint %}

* **distinct\_target\_key=None** - Укажите, следует ли при нетерпеливой загрузке подзапроса **subquery** применять ключевое слово **DISTINCT** к самому внутреннему оператору **SELECT**. Если оставить значение `None`, ключевое слово **DISTINCT** будет применяться в тех случаях, когда целевые столбцы не содержат полный первичный ключ целевой таблицы. Если установлено значение `True`, ключевое слово **DISTINCT** безоговорочно применяется к самому внутреннему **SELECT**. Может быть желательно установить этот флаг в `False`, когда **DISTINCT** снижает производительность самого внутреннего подзапроса сверх того, что может быть вызвано дублированием самых внутренних строк.

{% hint style="warning" %}
**Изменено в версии 0.9.0**: - **relationship.distinct\_target\_key** теперь по умолчанию имеет значение `None`, так что эта функция автоматически включается в тех случаях, когда самый внутренний запрос нацелен на неуникальный ключ.
{% endhint %}

{% hint style="info" %}
Смотри также:

[Relationship Loading Techniques](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html) - включает введение в нетерпеливую загрузку подзапросов.
{% endhint %}

* **doc** - Строка документации, которая будет применена к результирующему дескриптору.
* **foreign\_keys** - Список столбцов, которые должны использоваться в качестве столбцов «внешнего ключа» или столбцов, которые ссылаются на значение в удаленном столбце, в контексте условия **relationship.primaryjoin** этого объекта [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for). То есть, если условие **relationship.primaryjoin** этого отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) является `a.id == b.a_id`, и значения в **b.a\_id** должны присутствовать в **a.id**, то столбец «внешний ключ» этого отношение [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) — это **b.a\_id**. В обычных случаях параметр **relationship.foreign\_keys** _**не требуется**_. Отношение [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) автоматически определяет, какие столбцы в условии **relationship.primaryjoin** должны считаться столбцами «внешнего ключа» на основе тех объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), которые указывают [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey), или иным образом перечислены в качестве ссылающихся столбцов в конструкции [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint). **relationship.foreign\_keys** требуется только в том случае, если:
  1. Существует несколько способов создания соединения локальной таблицы с удаленной таблицей, поскольку существует несколько ссылок на внешние ключи. Установка **foreign\_keys** ограничит отношение [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), чтобы рассматривать только те столбцы, которые указаны здесь как «внешние».
  2. Сопоставляемая таблица [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) на самом деле не имеет конструкций [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey) или [ForeignKeyConstraint](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKeyConstraint), часто потому, что таблица была отражена из базы данных, которая не поддерживает отражение внешнего ключа (MySQL MyISAM).
  3. Аргумент **relationship.primaryjoin** используется для создания нестандартного условия соединения, в котором используются столбцы или выражения, которые обычно не ссылаются на их «родительский» столбец, например, условие соединения, выраженное сложным сравнением с использованием функции SQL.

Конструкция [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) будет выдавать информативные сообщения об ошибках, которые предлагают использовать параметр **relationship.foreign\_keys**, когда представлено неоднозначное условие. В типичных случаях, если отношение [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) не вызывает никаких исключений, параметр **relationship.foreign\_keys** обычно не требуется.

Отношение **relationship.foreign\_keys** также может быть передано как вызываемая функция, которая оценивается во время инициализации преобразователя, и может быть передана как строка, оцениваемая Python, при использовании декларативного.

{% hint style="danger" %}
При передаче в виде строки, которую может вычислить Python, аргумент интерпретируется с помощью функции Python _**eval()**_. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ**. Подробную информацию о декларативной оценке аргументов отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

{% hint style="info" %}
Смотри также:

[Handling Multiple Join Paths](nastroika-soedineniya-join-otnoshenii.md#obrabotka-neskolkikh-putei-soedineniya)

[Creating Custom Foreign Conditions](nastroika-soedineniya-join-otnoshenii.md#sozdanie-polzovatelskikh-vneshnikh-uslovii)

[`foreign()`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.foreign) - позволяет напрямую аннотировать «foreign» столбцы внутри [`relationship.primaryjoin`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) условия.
{% endhint %}

* **info** - Необязательный словарь данных, который будет заполнен атрибутом [MapperProperty.info](https://docs.sqlalchemy.org/en/14/orm/internals.html#sqlalchemy.orm.MapperProperty.info) этого объекта.
* **innerjoin=False** - Когда установлено значение `True`, объединенные нетерпеливые загрузки будут использовать внутреннее соединение для соединения со связанными таблицами вместо внешнего соединения. Целью этого параметра обычно является повышение производительности, поскольку внутренние соединения обычно работают лучше, чем внешние. Этот флаг может быть установлен в значение `True`, когда отношение ссылается на объект через «многие к одному» с использованием локальных внешних ключей, которые не могут принимать значение `NULL`, или когда ссылка является «один к одному» или коллекция, которая гарантированно имеет одну или по крайней мере одну запись. Параметр поддерживает те же «вложенные» и «невложенные» параметры, что и параметр [joinedload.innerjoin](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload.params.innerjoin). См. этот флаг для получения подробной информации о вложенном/невложенном поведении.

{% hint style="info" %}
Смотри также:

[`joinedload.innerjoin`](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.joinedload.params.innerjoin) - параметр, указанный параметром загрузчика, включая сведения о поведении вложения.

[What Kind of Loading to Use ?](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#what-kind-of-loading) - Обсуждение некоторых деталей различных вариантов загрузчика.
{% endhint %}

* **join\_depth** - Если не-None, целочисленное значение, указывающее, сколько уровней глубоких «нетерпеливых» загрузчиков должны присоединиться к самореферентной или циклической связи. Число показывает, сколько раз один и тот же преобразователь должен присутствовать в условии загрузки на определенной ветке соединения. Если оставить по умолчанию `None`, нетерпеливые загрузчики прекратят цепочку, когда они столкнутся с тем же целевым mapper, который уже находится выше в цепочке. Эта опция применима как к загрузчикам с присоединением, так и к подзапросам.

{% hint style="info" %}
Смотри также:

[Configuring Self-Referential Eager Loading](otnosheniya-spiska-smezhnosti.md#nastroika-samoreferentnoi-zhadnoi-zagruzki) - Вводная документация и примеры.
{% endhint %}

* **lazy='select'** - указывает, как должны быть загружены связанные элементы. По умолчанию значение **select**. Значения включают:
  * **select** - элементы следует загружать лениво при первом доступе к свойству, используя отдельный оператор **SELECT** или выборку карты идентификаторов для простых ссылок «многие к одному».
  * **immediate** - элементы следует загружать по мере загрузки родителей, используя отдельный оператор **SELECT** или выборку карты идентификаторов для простых ссылок «многие к одному».
  * **joined** - элементы должны быть загружены «с нетерпением» (_**eagerly**_) в том же запросе, что и родительский, с использованием **JOIN** или **LEFT OUTER JOIN**. Является ли соединение «внешним» или нет, определяется параметром **relationship.innerjoin**.
  * **subquery** - элементы следует загружать «с нетерпением» (_**eagerly**_) по мере загрузки родителей, используя один дополнительный оператор SQL, который выполняет **JOIN** для подзапроса исходного оператора для каждой запрошенной коллекции.
  * **selectin** - элементы следует загружать «с нетерпением» (_**eagerly**_) по мере загрузки родителей, используя один или несколько дополнительных операторов SQL, которые выполняют **JOIN** для непосредственного родительского объекта, указывая идентификаторы первичного ключа с помощью предложения **IN**. _**Новое в версии 1.2**_.
  * **noload** - никакая загрузка не должна происходить в любое время. Это делается для поддержки атрибутов «только для записи» (_**write-only**_) или атрибутов, которые заполняются каким-либо образом, специфичным для приложения.
  * **raise** - ленивая загрузка запрещена; доступ к атрибуту, если его значение еще не было загружено с помощью нетерпеливой загрузки, вызовет ошибку [InvalidRequestError](https://docs.sqlalchemy.org/en/14/core/exceptions.html#sqlalchemy.exc.InvalidRequestError). Эту стратегию можно использовать, когда объекты должны быть отсоединены от прикрепленного к ним сеанса [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) после их загрузки. _**Новое в версии 1.1**_.
  * **raise\_on\_sql** - ленивая загрузка, выдающая SQL, запрещена; доступ к атрибуту, если его значение еще не было загружено с помощью нетерпеливой загрузки, вызовет [InvalidRequestError](https://docs.sqlalchemy.org/en/14/core/exceptions.html#sqlalchemy.exc.InvalidRequestError), **если ленивая загрузка должна выдать SQL**. Если отложенная загрузка может извлечь связанное значение из карты идентификаторов или определить, что оно должно быть `None`, значение загружается. Эту стратегию можно использовать, когда объекты останутся связанными с прикрепленным сеансом [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), однако дополнительные операторы **SELECT** должны быть заблокированы. _**Новое в версии 1.1**_.
  * **dynamic** - атрибут будет возвращать предварительно настроенный объект запроса [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) для всех операций чтения, к которому можно применить дальнейшие операции фильтрации перед итерацией результатов. Дополнительные сведения см. в разделе [Загрузчики динамических отношений](konfiguraciya-kollekcii-i-metody-raboty-s-nimi/rabota-s-bolshimi-kollekciyami.md#zagruzchiki-dinamicheskikh-otnoshenii).
  * **True** - синоним **'select'**.
  * **False** - синоним **'joined'**.
  * **None** - синоним **'noload'**.

{% hint style="info" %}
Смотри также:

[Relationship Loading Techniques](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html) - Полная документация по настройке загрузчика отношений.

[Dynamic Relationship Loaders](konfiguraciya-kollekcii-i-metody-raboty-s-nimi/rabota-s-bolshimi-kollekciyami.md#zagruzchiki-dinamicheskikh-otnoshenii) - подробно о динамическом варианте.

[Setting Noload, RaiseLoad](konfiguraciya-kollekcii-i-metody-raboty-s-nimi/rabota-s-bolshimi-kollekciyami.md#nastroika-noload-raiseload) - заметки о “noload” и “raise”
{% endhint %}

* **load\_on\_pending=False** - Указывает поведение загрузки для временных или ожидающих родительских объектов. Если установлено значение `True`, отложенный загрузчик выдает запрос для родительского объекта, который не является постоянным, то есть он никогда не очищался. Это может повлиять на ожидающий объект, когда автосброс отключен, или на временный объект, который был «**attached**» к сеансу [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session), но не является частью его ожидающей коллекции. Флаг отношения **relationship.load\_on\_pending** не улучшает поведение при обычном использовании ORM — ссылки на объекты должны создаваться на уровне объекта, а не на уровне внешнего ключа, чтобы они присутствовали обычным образом до того, как начнется сброс. Этот флаг не предназначен для общего использования.

{% hint style="info" %}
Смотри также:

[`Session.enable_relationship_loading()`](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.enable\_relationship\_loading) - этот метод устанавливает поведение «загрузка в ожидании» (`load_on_pending`) для всего объекта, а также позволяет загружать объекты, которые остаются временными или отсоединенными.
{% endhint %}

* **order\_by** - Указывает порядок, который следует применять при загрузке этих элементов. Отношение **relationship.order\_by**, как ожидается, будет ссылаться на один из объектов [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column), с которым сопоставляется целевой класс, или на сам атрибут, привязанный к целевому классу, который ссылается на столбец. Отношение **relationship.order\_by** также может быть передано как вызываемая функция, которая оценивается во время инициализации преобразователя mapper, и может быть передана как оцениваемая Python строка при использовании Declarative.

{% hint style="danger" %}
При передаче в виде строки, которую может вычислить Python, аргумент интерпретируется с помощью функции Python _**eval()**_. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ**. Подробную информацию о декларативной оценке аргументов отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

* **passive\_deletes=False** - Указывает поведение загрузки во время операций удаления. Значение `True` указывает, что выгруженные дочерние элементы не должны загружаться во время операции удаления родительского элемента. Обычно при удалении родительского элемента загружаются все дочерние элементы, чтобы их можно было либо пометить как удаленные, либо установить их внешний ключ к родительскому элементу равным `NULL`. Пометка этого флага как `True` обычно подразумевает наличие правила `ON DELETE <CASCADE|SET NULL>`, которое будет обрабатывать обновление/удаление дочерних строк на стороне базы данных. Кроме того, установка флага в строковое значение **«all»** отключит «обнуление» дочерних внешних ключей, когда родительский объект удален и не включен каскад **delete** или **delete-orphan**. Обычно это используется, когда на стороне базы данных имеется сценарий запуска или возникновения ошибки. Обратите внимание, что атрибуты внешнего ключа в дочерних объектах в сеансе не будут изменены после сброса, поэтому это очень особая настройка варианта использования. Кроме того, «обнуление» все равно будет происходить, если дочерний объект деассоциирован с родителем.

{% hint style="info" %}
Смотри также:

[Using foreign key ON DELETE cascade with ORM relationships](https://docs.sqlalchemy.org/en/14/orm/cascades.html#passive-deletes) - Вводная документация и примеры.
{% endhint %}

* **passive\_update=False** - Указывает поведение сохраняемости, которое должно выполняться, когда ссылочное значение первичного ключа изменяется на месте, указывая, что для ссылочных столбцов внешнего ключа также потребуется изменить свое значение. При значении `True` предполагается, что **ON UPDATE CASCADE** настроен для внешнего ключа в базе данных и что база данных будет обрабатывать распространение **UPDATE** из исходного столбца в зависимые строки. При значении `False` конструкция SQLAlchemy [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) попытается создать собственные операторы **UPDATE** для изменения связанных целей. Однако обратите внимание, что SQLAlchemy **не может** генерировать **UPDATE** для более чем одного уровня каскада. Кроме того, установка этого флага в `False` несовместима в случае, когда база данных фактически обеспечивает ссылочную целостность, если только эти ограничения явно не «отложены» (_deferred_), если целевой сервер поддерживает это. Настоятельно рекомендуется, чтобы приложение, использующее изменяемые первичные ключи, сохраняло для параметра **passive\_updates** значение `True` и вместо этого использовало функции ссылочной целостности самой базы данных для эффективной и полной обработки изменений.

{% hint style="info" %}
Смотри также:

[Mutable Primary Keys / Update Cascades](shablony-sokhraneniya-osobykh-otnoshenii.md#izmenyaemye-pervichnye-klyuchi-kaskady-obnovlenii) - Вводная документация и примеры.

[`mapper.passive_updates`](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.mapper.params.passive\_updates) - аналогичный флаг, который вступает в силу для сопоставлений наследования объединенных таблиц.
{% endhint %}

* **post\_update** - Это указывает на то, что связь должна обрабатываться вторым оператором **UPDATE** после **INSERT** или перед **DELETE**. В настоящее время он также выдает **UPDATE** после того, как экземпляр был **UPDATed**, хотя технически это должно быть улучшено. Этот флаг используется для сохранения двунаправленных зависимостей между двумя отдельными строками (т. е. каждая строка ссылается на другую), где в противном случае было бы невозможно полностью **INSERT** или **DELETE** обе строки, поскольку одна строка существует раньше другой. Используйте этот флаг, когда конкретная схема сопоставления будет включать две строки, которые зависят друг от друга, например, таблица, которая имеет отношение «один ко многим» к набору дочерних строк, а также имеет столбец, который ссылается на одну дочернюю строку внутри этого списка (т.е. обе таблицы содержат внешний ключ друг к другу). Если операция сброса возвращает ошибку об обнаружении «циклической зависимости», это сигнал о том, что вы можете захотеть использовать **relationship.post\_update** для «разрыва» цикла.

{% hint style="info" %}
Смотри также:

[Rows that point to themselves / Mutually Dependent Rows](shablony-sokhraneniya-osobykh-otnoshenii.md#stroki-kotorye-ukazyvayut-sami-na-sebya-vzaimozavisimye-stroki) - Вводная документация и примеры.
{% endhint %}

*   **primaryjoin** - Выражение SQL, которое будет использоваться в качестве основного соединения дочернего объекта с родительским объектом или в отношении "многие ко многим" для соединения родительского объекта с ассоциативной таблицей. По умолчанию это значение вычисляется на основе отношений внешнего ключа родительской и дочерней таблиц (или ассоциативной таблицы).

    **relationship.primaryjoin** также может быть передано как вызываемая функция, которая оценивается во время инициализации преобразователя, и может быть передана как оцениваемая Python строка при использовании **Declarative**.

{% hint style="danger" %}
При передаче в виде строки, которую может вычислить Python, аргумент интерпретируется с помощью функции Python _**eval()**_. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ**. Подробную информацию о декларативной оценке аргументов отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

{% hint style="info" %}
Смотри также:

[Specifying Alternate Join Conditions](nastroika-soedineniya-join-otnoshenii.md#ukazanie-alternativnykh-uslovii-obedineniya-join)
{% endhint %}

*   **remote\_side** - Используется для самореферентных отношений, указывает столбец или список столбцов, формирующих «удаленную сторону» отношения.

    **relationship.remote\_side** также может быть передано как вызываемая функция, которая оценивается во время инициализации преобразователя, и может быть передана как строка, оцениваемая Python, при использовании **Declarative**.

{% hint style="danger" %}
При передаче в виде строки, которую может вычислить Python, аргумент интерпретируется с помощью функции Python _**eval()**_. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ**. Подробную информацию о декларативной оценке аргументов отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

{% hint style="info" %}
Смотри также:

[Adjacency List Relationships](otnosheniya-spiska-smezhnosti.md) - подробное объяснение того, как [`relationship.remote_side`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.remote\_side) используется для настройки самореферентных отношений.

[`remote()`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.remote) - функция аннотации, которая выполняет ту же цель, что и [`relationship.remote_side`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.remote\_side), обычно используется, когда пользовательское условие  [`relationship.primaryjoin`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.primaryjoin) применяется.
{% endhint %}

* **query\_class** - Подкласс [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query), который будет использоваться внутри **AppenderQuery**, возвращенный «динамическим» отношением, то есть отношением, которое указывает **`lazy="dynamic"`** или было создано иным образом с использованием функции [dynamic\_loader()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.dynamic\_loader).

{% hint style="info" %}
Смотри также:

[Dynamic Relationship Loaders](konfiguraciya-kollekcii-i-metody-raboty-s-nimi/rabota-s-bolshimi-kollekciyami.md#zagruzchiki-dinamicheskikh-otnoshenii) - Введение в «динамические» загрузчики отношений.
{% endhint %}

*   **secondary\_join** - Выражение SQL, которое будет использоваться в качестве соединения таблицы ассоциаций с дочерним объектом. По умолчанию это значение вычисляется на основе отношений внешнего ключа ассоциации и дочерних таблиц.

    **relationship.secondaryjoin** также может быть передано как вызываемая функция, которая оценивается во время инициализации преобразователя, и может быть передана как оцениваемая Python строка при использовании **Declarative**.

{% hint style="danger" %}
При передаче в виде строки, которую может вычислить Python, аргумент интерпретируется с помощью функции Python _**eval()**_. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ**. Подробную информацию о декларативной оценке аргументов отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) см. в разделе Оценка аргументов отношений ([Evaluation of relationship arguments](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/relationships.html#declarative-relationship-eval)).
{% endhint %}

{% hint style="info" %}
Смотри также:

[Specifying Alternate Join Conditions](nastroika-soedineniya-join-otnoshenii.md#ukazanie-alternativnykh-uslovii-obedineniya-join)
{% endhint %}

* **single\_parent** - При значении `True` устанавливает валидатор, который не позволит объектам быть связанными более чем с одним родителем одновременно. Это используется для отношений «многие к одному» или «многие ко многим», которые следует рассматривать либо как «один к одному», либо как «один ко многим». Его использование является необязательным, за исключением конструкций отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), которые являются много-к-одному или многие-ко-многим, а также указывают опцию каскадного **delete-orphan**. Сама конструкция отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) вызовет ошибку, указывающую, когда требуется эта опция.

{% hint style="info" %}
Смотри также:

[Cascades](https://docs.sqlalchemy.org/en/14/orm/cascades.html#unitofwork-cascades) - содержит подробную информацию о том, когда флаг [`relationship.single_parent`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.single\_parent) может быть уместным.
{% endhint %}

*   **uselist** - Логическое значение, указывающее, следует ли загружать это свойство как список или скаляр. В большинстве случаев это значение определяется автоматически с помощью отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) во время настройки преобразователя mapper в зависимости от типа и направления отношения — **один ко многим** формирует **список**, **многие к одному** формируют **скаляр**, **многие ко многим** — **список**. Если требуется скаляр там, где обычно должен присутствовать список, например, двунаправленное отношение один к одному, установите для **relationship.uselist** значение `False`.

    Флаг **relationship.uselist** также доступен в существующей конструкции отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) в качестве атрибута только для чтения, который можно использовать, чтобы определить, имеет ли это отношение [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) отношение к коллекциям или скалярным атрибутам:

```python
>>> User.addresses.property.uselist
True
```

{% hint style="info" %}
Смотри также:

[One To One](bazovye-shablony-relationship.md#odin-ko-odnomu-one-to-one) - Знакомство с моделью отношений «один к одному», которая обычно требует флаг [`relationship.uselist`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.uselist).
{% endhint %}

*   **viewonly=False** - Если установлено значение `True`, отношение используется только для загрузки объектов, а не для какой-либо операции сохранения. Отношение [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), в котором указано **relationship.viewonly**, может работать с более широким диапазоном операций SQL в рамках условия **relationship.primaryjoin**, включая операции, в которых используются различные операторы сравнения, а также функции SQL, такие как [cast()](https://docs.sqlalchemy.org/en/14/core/sqlelement.html#sqlalchemy.sql.expression.cast). Флаг **relationship.viewonly** также часто используется при определении любого вида отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for), которое не представляет полный набор связанных объектов, чтобы предотвратить изменения коллекции, приводящие к операциям персистентности.

    При использовании флага **relationship.viewonly** в сочетании с обратными ссылками исходная связь для определенного изменения состояния не будет производить изменения состояния в отношении только для просмотра. Это поведение подразумевается тем, что для свойства **relationship.sync\_backref** задано значение `False`.

{% hint style="warning" %}
**Изменено в версии 1.3.17**: - флаг **relationship.sync\_backref** устанавливается в `False` при использовании только просмотра в сочетании с обратными ссылками.
{% endhint %}

{% hint style="info" %}
Смотри также:

[`relationship.sync_backref`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.sync\_backref)
{% endhint %}

*   **sync\_backref** - Логическое значение, которое включает события, используемые для синхронизации атрибутов Python, когда это отношение является целью **relationship.backref** или **relationship.back\_populates**.

    По умолчанию установлено значение `None`, что указывает на то, что следует выбирать автоматическое значение на основе значения флага **relationship.viewonly**. Если оставить значение по умолчанию, изменения в состоянии будут повторно заполнены только в том случае, если ни одна из сторон отношения не доступна только для просмотра.

{% hint style="warning" %}
**Новое в версии 1.3.17**.
{% endhint %}

{% hint style="warning" %}
**Изменено в версии 1.4**: — Отношение, указывающее, что **relationship.viewonly**, автоматически подразумевает, что для **relationship.sync\_backref** установлено значение `False`.
{% endhint %}

{% hint style="info" %}
Смотри также:

[`relationship.viewonly`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship.params.viewonly)
{% endhint %}

* **omit\_join** - Позволяет вручную управлять оптимизацией автоматического соединения **selectin**. Установите значение `False`, чтобы отключить функцию «**omit\_join**», добавленную в SQLAlchemy 1.3; или оставьте значение `None`, чтобы оставить автоматическую оптимизацию.

{% hint style="info" %}
Этот флаг может быть установлен только в **False**. Нет необходимости устанавливать значение **True**, так как оптимизация «**omit\_join**» определяется автоматически; если он не обнаружен, то оптимизация не поддерживается.
{% endhint %}

{% hint style="warning" %}
Новое в версии 1.3.
{% endhint %}

{% hint style="warning" %}
**Изменено в версии 1.3.11**: при установке **omit\_join** на **True** теперь будет выдаваться предупреждение, так как это не было предполагаемым использованием этого флага.
{% endhint %}

### _function_ sqlalchemy.orm.backref(_name_, _\*\*kwargs_)

Создает обратную ссылку с явными аргументами ключевого слова, которые являются теми же самыми аргументами, которые можно отправить в [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for).

Используется с аргументом ключевого слова **backref** для отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) вместо строкового аргумента, например:

```python
'items':relationship(
    SomeItem, backref=backref('parent', lazy='subquery'))
```

{% hint style="info" %}
Смотри также:

[Linking Relationships with Backref](svyazyvanie-otnoshenii-s-backref.md#svyazyvanie-otnoshenii-s-backref)
{% endhint %}

### _function_ sqlalchemy.orm.relation(_\*arg_, _\*\*kw_)

Синоним [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for).

{% hint style="danger" %}
**Устарело, начиная с версии 1.4**: конструкция отношения считается устаревшей, начиная с серии 1.x SQLAlchemy, и будет удалена в версии 2.0. Пожалуйста, используйте [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for). (Основные сведения о SQLAlchemy 2.0 в: Миграция на SQLAlchemy 2.0 ([Migrating to SQLAlchemy 2.0](https://docs.sqlalchemy.org/en/14/changelog/migration\_20.html)))
{% endhint %}

### _function_ sqlalchemy.orm.dynamic\_loader(_argument_, _\*\*kw_)

Создайте динамически загружаемое свойство сопоставления mapper.

По сути, это то же самое, что и использование аргумента **lazy='dynamic'** для отношения [relationship()](relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for):

```python
dynamic_loader(SomeClass)

# такой же как

relationship(SomeClass, lazy="dynamic")
```

Дополнительные сведения о динамической загрузке см. в разделе [Загрузчики динамических отношений](konfiguraciya-kollekcii-i-metody-raboty-s-nimi/rabota-s-bolshimi-kollekciyami.md#zagruzchiki-dinamicheskikh-otnoshenii).

### _function_ sqlalchemy.orm.foreign(_expr_)

Аннотирует часть выражения первичного соединения «внешней» аннотацией.

Описание использования см. в разделе [Создание пользовательских внешних условий](nastroika-soedineniya-join-otnoshenii.md#sozdanie-polzovatelskikh-vneshnikh-uslovii).

{% hint style="info" %}
Смотри также:

[Creating Custom Foreign Conditions](nastroika-soedineniya-join-otnoshenii.md#sozdanie-polzovatelskikh-vneshnikh-uslovii)

[`remote()`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.remote)
{% endhint %}

### _function_ sqlalchemy.orm.remote(_expr_)

Аннотирует часть выражения первичного соединения «удаленной» аннотацией.

Описание использования см. в разделе [Создание пользовательских внешних условий](nastroika-soedineniya-join-otnoshenii.md#sozdanie-polzovatelskikh-vneshnikh-uslovii).

{% hint style="info" %}
Смотри также:

[Creating Custom Foreign Conditions](nastroika-soedineniya-join-otnoshenii.md#sozdanie-polzovatelskikh-vneshnikh-uslovii)

[`foreign()`](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.foreign)
{% endhint %}
