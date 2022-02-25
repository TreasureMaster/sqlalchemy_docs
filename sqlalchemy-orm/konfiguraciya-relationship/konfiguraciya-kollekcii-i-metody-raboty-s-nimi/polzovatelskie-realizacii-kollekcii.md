# Пользовательские реализации коллекций

Вы также можете использовать свои собственные типы для коллекций. В простых случаях наследование от списка **list** или множества **set**, добавление пользовательского поведения — это все, что нужно. В других случаях необходимы специальные декораторы, чтобы более подробно сообщить SQLAlchemy о том, как работает коллекция.

{% hint style="info" %}
**Нужна ли мне реализация пользовательской коллекции?**

В большинстве случаев вовсе не нужна! Наиболее распространенными вариантами использования настраиваемой **custom** коллекции являются те, которые проверяют или упорядочивают входящие значения в новую форму, например строку, которая становится экземпляром класса, или тот, который идет дальше и представляет данные каким-то образом внутри, представляет эти данные снаружи другой формы.

Для первого варианта использования декоратор [validates()](../../konfiguraciya-mapper/otobrazhenie-kolonok-i-vyrazhenii/izmenenie-povedeniya-atributov.md#function-sqlalchemy.orm.validates-names-kw) — это, безусловно, самый простой способ перехвата входящих значений во всех случаях для целей проверки и простого маршалинга. См. [Простые валидаторы](../../konfiguraciya-mapper/otobrazhenie-kolonok-i-vyrazhenii/izmenenie-povedeniya-atributov.md#prostye-validatory) для примера.

Для второго варианта использования расширение [Association Proxy](https://docs.sqlalchemy.org/en/14/orm/extensions/associationproxy.html) представляет собой хорошо протестированную, широко используемую систему, которая обеспечивает чтение/запись представления **view** коллекции с точки зрения некоторого атрибута, присутствующего в целевом объекте. Поскольку целевым атрибутом может быть `@property`, возвращающее практически что угодно, широкий спектр «альтернативных» представлений коллекции может быть создан с помощью всего нескольких функций. Этот подход не затрагивает базовую отображаемую коллекцию и устраняет необходимость тщательной настройки поведения коллекции для каждого метода.

Настраиваемые коллекции полезны, когда коллекция должна иметь особое поведение при операциях доступа или изменения, которые иначе нельзя смоделировать вне коллекции. Конечно, их можно комбинировать с двумя вышеуказанными подходами.
{% endhint %}

Коллекции в SQLAlchemy прозрачно _инструментированы_. Инструментирование означает, что нормальные операции над коллекцией отслеживаются и приводят к записи изменений в базу данных во время сброса. Кроме того, операции сбора могут запускать _**события**_, указывающие на то, что должна быть выполнена какая-то вторичная операция. Примеры вторичной операции включают сохранение дочернего элемента в сеансе  [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) родителя (т. е. каскад _**save-update**_), а также синхронизацию состояния двунаправленной связи (т. е. [backref()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.backref)).

Пакет **collections** понимает базовый интерфейс списков, наборов и словарей и автоматически применяет инструментарий к этим встроенным типам и их подклассам. Типы, производные от объектов, которые реализуют базовый интерфейс коллекции, обнаруживаются и инструментируются с помощью утиного ввода:

```python
class ListLike(object):
    def __init__(self):
        self.data = []
    def append(self, item):
        self.data.append(item)
    def remove(self, item):
        self.data.remove(item)
    def extend(self, items):
        self.data.extend(items)
    def __iter__(self):
        return iter(self.data)
    def foo(self):
        return 'foo'
```

_**append**_, _**remove**_ и _**extend**_ являются известными спископодобными методами и будут обрабатываться автоматически. **\_\_iter\_\_** не является методом-мутатором и не будет инструментирован, как и _**foo**_.

Утиная типизация (т. е. догадки), конечно, не является надежной, поэтому вы можете четко указать интерфейс, который вы реализуете, предоставив атрибут класса **\_\_emulates\_\_**:

```python
class SetLike(object):
    __emulates__ = set

    def __init__(self):
        self.data = set()
    def append(self, item):
        self.data.add(item)
    def remove(self, item):
        self.data.remove(item)
    def __iter__(self):
        return iter(self.data)
```

Этот класс выглядит как список из-за добавления _**append**_, но **\_\_emulates\_\_** заставляет его быть похожим на множество. Известно, что _**remove**_ является частью интерфейса **set** и будет инструментирован.

Но этот класс пока не будет работать: нужно немного клея, чтобы адаптировать его для использования SQLAlchemy. ORM необходимо знать, какие методы использовать для добавления, удаления и повторения элементов коллекции. При использовании таких типов, как **list** или **set**, соответствующие методы хорошо известны и используются автоматически, если они присутствуют. Этот похожий на набор класс не предоставляет ожидаемого метода добавления _**add**_, поэтому мы должны предоставить явное отображение для ORM через декоратор.

## Аннотирование пользовательских коллекций с помощью декораторов

Декораторы можно использовать для пометки отдельных методов, необходимых ORM для управления коллекциями. Используйте их, когда ваш класс не совсем соответствует обычному интерфейсу для своего типа контейнера или когда вы хотели бы использовать другой метод для выполнения работы.

```python
from sqlalchemy.orm.collections import collection

class SetLike(object):
    __emulates__ = set

    def __init__(self):
        self.data = set()

    @collection.appender
    def append(self, item):
        self.data.add(item)

    def remove(self, item):
        self.data.remove(item)

    def __iter__(self):
        return iter(self.data)
```

И это все, что нужно для завершения примера. SQLAlchemy добавит экземпляры с помощью метода _**append**_. _**remove**_ и **\_\_iter\_\_** являются методами по умолчанию для наборов и будут использоваться для удаления и итерации. Методы по умолчанию также можно изменить:

```python
from sqlalchemy.orm.collections import collection

class MyList(list):
    @collection.remover
    def zark(self, item):
        # do something special...

    @collection.iterator
    def hey_use_this_instead_for_iteration(self):
        # ...
```

Совсем не обязательно быть похожим на список или множество. Классы коллекций могут быть любой формы, если они имеют интерфейс добавления, удаления и итерации, помеченный для использования SQLAlchemy. Методы добавления и удаления будут вызываться с сопоставленным объектом в качестве единственного аргумента, а методы итератора вызываются без аргументов и должны возвращать итератор.

| Название объекта                                                                                           | Описание                                    |
| ---------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| [collection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.collection) | Декораторы для классов коллекций сущностей. |

### _class_ sqlalchemy.orm.collections.collection

Декораторы для классов коллекций сущностей.

Декораторы делятся на две группы: аннотации и рецепты перехвата.

Аннотирующие декораторы (**appender**, **remover**, **iterator**, **convert**, **internally\_instrumented**) указывают цель метода и не принимают аргументов. Они не пишутся со скобками:

```python
@collection.appender
def append(self, append): ...
```

Все декораторы рецептов требуют круглых скобок, даже те, которые не принимают аргументов:

```python
@collection.adds('entity')
def insert(self, position, entity): ...

@collection.removes_return()
def popitem(self): ...
```

### _method_ sqlalchemy.orm.collections.collection._static_ adds (_arg_)

Отметьте метод как добавление объекта в коллекцию.

Добавляет в метод обработку «добавить в коллекцию». Аргумент декоратора **arg** указывает, какой аргумент метода содержит значение, относящееся к SQLAlchemy. Аргументы могут быть указаны позиционно (т.е. целыми числами) или по имени:

```python
@collection.adds(1)
def push(self, item): ...

@collection.adds('entity')
def do_stuff(self, thing, entity=None): ...
```

### _method_ sqlalchemy.orm.collections.collection._static_ appender(_fn_)

Пометьте метод как добавляющий к коллекции.

Метод **appender** вызывается с одним позиционным аргументом: добавляемым значением. Метод будет автоматически украшен `'adds(1)'`, если он еще не оформлен:

```python
@collection.appender
def add(self, append): ...

# или, что то же самое
@collection.appender
@collection.adds(1)
def add(self, append): ...

# для типа сопоставления «append» может удалить предыдущее значение,
# занимающее этот слот. Рассмотрим d['a'] = 'foo' - любое предыдущее
# значение в d['a'] отбрасывается.
@collection.appender
@collection.replaces(1)
def add(self, entity):
    key = some_key_func(entity)
    previous = None
    if key in self:
        previous = self[key]
    self[key] = entity
    return previous
```

Если добавляемое значение не разрешено в коллекции, вы можете создать исключение. Следует помнить, что добавление будет вызываться для каждого объекта, сопоставленного запросом к базе данных. Если в базе данных есть строки, которые нарушают семантику вашей коллекции, вам придется проявить изобретательность, чтобы исправить проблему, так как доступ через коллекцию работать не будет.

Если метод **appender** внутренне инструментирован, вы также должны получить аргумент ключевого слова **«\_sa\_initiator»** и обеспечить его обнародование в событиях сбора.

### _method_ sqlalchemy.orm.collections.collection._static_ converter(_fn_)

Помечает метод как преобразователь коллекции.

{% hint style="danger" %}
**Устарело, начиная с версии 1.3**: обработчик [collection.converter()](polzovatelskie-realizacii-kollekcii.md#method-sqlalchemy.orm.collections.collection.static-converter-fn) устарел и будет удален в будущем выпуске. Обратитесь к интерфейсу слушателя **bulk\_replace** в сочетании с функцией [listen()](https://docs.sqlalchemy.org/en/14/core/event.html#sqlalchemy.event.listen).
{% endhint %}

Этот необязательный метод будет вызываться при полной замене коллекции, например:

```python
myobj.acollection = [newvalue1, newvalue2]
```

Метод **converter** получит присваиваемый объект и должен вернуть итерацию значений, подходящих для использования методом добавления **appender**. Преобразователь **converter** не должен присваивать значения или изменять коллекцию, его единственная задача — адаптировать значение, предоставленное пользователем, в итерацию значений для использования ORM.

Реализация **converter** по умолчанию будет использовать утиную печать для преобразования. Коллекция, похожая на словарь, будет преобразована в итерацию значений словаря, а другие типы будут просто итерированы:

```python
@collection.converter
def convert(self, other): ...
```

Если утиная типизация объекта не соответствует типу этой коллекции, возникает ошибка **TypeError**.

Предоставьте реализацию этого метода, если вы хотите расширить диапазон возможных типов, которые можно назначать массово, или выполнить проверку значений, которые должны быть назначены.

### _method_ sqlalchemy.orm.collections.collection._static_ internally\_instrumented(_fn_)

Отмечает метод как инструментированный.

Этот тег предотвратит применение каких-либо декораторов к методу. Используйте это, если вы организуете свои собственные вызовы **collection\_adapter()** в одном из основных методов интерфейса SQLAlchemy или чтобы предотвратить автоматическое оформление метода **ABC** для вашей реализации:

```
# обычно метод 'extend' в спископодобном классе
# будет автоматически перехватываться и повторно
# реализовываться с точки зрения событий SQLAlchemy и append().
# Ваша реализация никогда не будет вызываться, если только:
@collection.internally_instrumented
def extend(self, items): ...
```

### _method_ sqlalchemy.orm.collections.collection._static_ iterator(_fn_)

Помечает метод как средство итерирования коллекции.

Метод **iterator** вызывается без аргументов. Ожидается, что он вернет итератор для всех членов коллекции:

```python
@collection.iterator
def __iter__(self): ...
```

### _method_ sqlalchemy.orm.collections.collection._static_ remover(_fn_)

Помечает метод как средство удаления коллекции.

Метод удаления вызывается с одним позиционным аргументом: удаляемым значением. Метод будет автоматически декорирован с помощью [removes\_return()](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.collection.removes\_return), если он еще не оформлен:

```python
@collection.remover
def zap(self, entity): ...

# или, что то же самое
@collection.remover
@collection.removes_return()
def zap(self, ): ...
```

Если удаляемое значение отсутствует в коллекции, вы можете создать исключение или вернуть `None`, чтобы игнорировать ошибку.

Если метод удаления внутренне инструментирован, вы также должны получить аргумент ключевого слова **«\_sa\_initiator»** и обеспечить его опубликование в событиях сбора.

### _method_ sqlalchemy.orm.collections.collection._static_ removes(_arg_)

Отмечает метод как удаляющий объект в коллекции.

Добавляет в метод обработку «удалить из коллекции». Аргумент **arg** декоратора указывает, какой аргумент метода содержит релевантное для SQLAlchemy значение, которое необходимо удалить. Аргументы могут быть указаны позиционно (т.е. целыми числами) или по имени:

```python
@collection.removes(1)
def zap(self, item): ...
```

Для методов, в которых удаляемое значение неизвестно во время вызова, используйте **collection.removes\_return**.

### _method_ sqlalchemy.orm.collections.collection._static_ removes\_return()

Отмечает метод как удаляющий объект в коллекции.

Добавляет в метод обработку «удалить из коллекции». Возвращаемое значение метода, если таковое имеется, считается удаляемым значением. Аргументы метода не проверяются:

```python
@collection.removes_return()
def pop(self): ...
```

Для методов, в которых удаляемое значение известно во время вызова, используйте **collection.remove**.

### _method_ sqlalchemy.orm.collections.collection._static_ replaces(_arg_)

Отмечает метод как замену сущности в коллекции.

Добавляет в метод обработку «добавить в коллекцию» и «удалить из коллекции». Аргумент **arg** декоратора указывает, какой аргумент метода содержит значение, относящееся к SQLAlchemy, которое нужно добавить, и возвращаемое значение, если оно будет считаться значением для удаления.

Аргументы могут быть указаны позиционно (т.е. целыми числами) или по имени:

```python
@collection.replaces(2)
def __setitem__(self, index, item): ...
```

## Пользовательские коллекции на основе словарей

Класс [MappedCollection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.MappedCollection) можно использовать в качестве базового класса для настраиваемых типов или в качестве дополнения для быстрого добавления поддержки коллекций **dict** в другие классы. Он использует ключевые функции для делегирования **\_\_setitem\_\_** и **\_\_delitem\_\_**:

```python
from sqlalchemy.util import OrderedDict
from sqlalchemy.orm.collections import MappedCollection

class NodeMap(OrderedDict, MappedCollection):
    """Содержит объекты «Node», отмеченные атрибутом «name»"""
    """с сохранением порядка вставки."""

    def __init__(self, *args, **kw):
        MappedCollection.__init__(self, keyfunc=lambda node: node.name)
        OrderedDict.__init__(self, *args, **kw)
```

При создании подкласса [MappedCollection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.MappedCollection) определяемые пользователем версии **\_\_setitem\_\_()** или **\_\_delitem\_\_()** должны быть дополнены [collection.internally\_instrumented()](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.collection.internally\_instrumented), если они вызывают те же самые методы в [MappedCollection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.MappedCollection). Это связано с тем, что методы в [MappedCollection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.MappedCollection) уже инструментированы — их вызов из уже инструментированного вызова может привести к многократному или ненадлежащему запуску событий, что в редких случаях приводит к повреждению внутреннего состояния:

```python
from sqlalchemy.orm.collections import MappedCollection,\
                                    collection

class MyMappedCollection(MappedCollection):
    """Используйте @internally_instrumented,"""
    """когда ваши методы вызывают уже инструментированные методы."""

    @collection.internally_instrumented
    def __setitem__(self, key, value, _sa_initiator=None):
        # делает что-то с key, value
        super(MyMappedCollection, self).__setitem__(key, value, _sa_initiator)

    @collection.internally_instrumented
    def __delitem__(self, key, _sa_initiator=None):
        # делает что-то с key
        super(MyMappedCollection, self).__delitem__(key, _sa_initiator)
```

ORM понимает интерфейс **dict** точно так же, как списки и наборы, и автоматически инструментирует все методы, подобные **dict**, если вы решите создать подкласс **dict** или обеспечить поведение коллекции, подобное **dict**, в классе с утиным типом. Однако вы должны декорировать методы добавления и удаления — в базовом интерфейсе словаря нет совместимых методов, которые SQLAlchemy может использовать по умолчанию. Итерация будет проходить через **itervalues()**, если не указано иное.

{% hint style="info" %}
Из-за ошибки в **MappedCollection** до версии 0.7.6 этот обходной путь обычно необходимо вызывать до того, как можно будет использовать пользовательский подкласс MappedCollection, который использует [collection.internally\_instrumented()](polzovatelskie-realizacii-kollekcii.md#method-sqlalchemy.orm.collections.collection.static-internally\_instrumented-fn):

```python
from sqlalchemy.orm.collections import _instrument_class, MappedCollection
_instrument_class(MappedCollection)
```

Это обеспечит правильную инициализацию **MappedCollection** с помощью пользовательских методов **\_\_setitem\_\_()** и **\_\_delitem\_\_()** перед использованием в пользовательском подклассе.
{% endhint %}

| Название объекта                                                                                                       | Описание                                   |
| ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| [MappedCollection](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.collections.MappedCollection) | Базовый класс коллекции на основе словаря. |

### _class_ sqlalchemy.orm.collections.MappedCollection(_keyfunc_)

Базовый класс коллекции на основе словаря.

Расширяет **dict** с помощью минимальной семантики пакетов, которая требуется классам коллекций. **set** и **remove** реализованы в терминах функции ключа: любой вызываемый объект, который принимает объект и возвращает объект для использования в качестве ключа словаря.

{% hint style="info" %}
Сигнатура класса:

class `sqlalchemy.orm.collections.MappedCollection` (`builtins.dict`)
{% endhint %}

### _method_ sqlalchemy.orm.collections.MappedCollection.\_\_init\_\_(_keyfunc_)

Создает новую коллекцию с ключом, предоставленным **keyfunc**.

**keyfunc** может быть любой вызываемой функцией, которая принимает объект и возвращает объект для использования в качестве ключа словаря. **Keyfunc** будет вызываться каждый раз, когда ORM необходимо добавить элемент только по значению (например, при загрузке экземпляров из базы данных) или удалить член. Обычные предупреждения о словарном ключе **apply-keyfunc(object)** должны возвращать один и тот же вывод на протяжении всего срока существования коллекции. Использование ключей на основе изменяемых свойств может привести к тому, что недоступные экземпляры будут «потеряны» в коллекции.

### _method_ sqlalchemy.orm.collections.MappedCollection.clear() → None

Удаляет все элементы из словаря.

### _method_ sqlalchemy.orm.collections.MappedCollection.pop(_k_\[, _d_]) → v

Удаляет указанный ключ и возвращает соответствующее значение.

Если ключ не найден, возвращается **d**, если он задан, в противном случае возникает **KeyError.**

### _method_ sqlalchemy.orm.collections.MappedCollection.popitem() → (k, v)

Удаляет и возвращает некоторую пару (ключ, значение) в виде кортежа с 2 значениями; но вызывает **KeyError**, если словарь пуст.

### _method_ sqlalchemy.orm.collections.MappedCollection.remove(_value_, _\_sa\_initiator=None_)

Удаляет элемент по значению, обратившись к **keyfunc** за ключом.

### _method_ sqlalchemy.orm.collections.MappedCollection.set(_value_, _\_sa\_initiator=None_)

Добавляет элемент по значению, обратившись к **keyfunc** для ключа.

### _method_ sqlalchemy.orm.collections.MappedCollection.setdefault(_key_, _default=None_)

Вставляет ключ со значением по умолчанию, если ключ отсутствует в словаре.

Возвращает значение для ключа, если ключ есть в словаре, иначе по умолчанию.

### _method_ sqlalchemy.orm.collections.MappedCollection.update(\[_E_, ]_\*\*F_) → None

Обновляет **D** из dict/iterable **E** и **F**.

Если **E** присутствует и имеет метод `.keys()`, то делает: для **k** в **E**: `D[k] = E[k]` Если **E** присутствует и не имеет метода `.keys()`, то делает: для `k, v` в **E**: `D[k] = v` В любом случае за этим следует: для **k** в **F**: `D[k] = F[k]`

## Инструментарий и пользовательские типы

Многие пользовательские типы и существующие библиотечные классы можно использовать в качестве типа коллекции сущностей без дальнейших церемоний. Однако важно отметить, что процесс инструментирования изменит тип, автоматически добавляя декораторы вокруг методов.

Декорации легки и не работают вне отношений, но они добавляют ненужные накладные расходы, когда срабатывают в другом месте. При использовании библиотечного класса в качестве коллекции может быть хорошей практикой использовать трюк с «тривиальным подклассом», чтобы ограничить декорации только вашим использованием в отношениях. Например:

```python
class MyAwesomeList(some.great.library.AwesomeList):
    pass

# ... relationship(..., collection_class=MyAwesomeList)
```

ORM использует этот подход для встроенных функций, тихо заменяя тривиальный подкласс, когда список, множество или словарь используются напрямую.
