# Работа с большими коллекциями

Поведение отношения [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) по умолчанию заключается в полной загрузке коллекции элементов в соответствии со стратегией загрузки отношения. Кроме того, сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) по умолчанию знает, как удалять только те объекты, которые фактически присутствуют в сеансе. Когда родительский экземпляр помечается для удаления и сбрасывается, сеанс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) загружает свой полный список дочерних элементов, так что они могут быть либо удалены, либо значение их внешнего ключа установлено равным нулю; это делается для того, чтобы избежать нарушений ограничений. Для больших коллекций дочерних элементов существует несколько стратегий обхода полной загрузки дочерних элементов как во время загрузки, так и во время удаления.

## Загрузчики динамических отношений

{% hint style="info" %}
SQLAlchemy 2.0 будет иметь немного измененный шаблон для «динамических» загрузчиков, который не зависит от объекта [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query), который будет унаследован в версии 2.0. Текущие стратегии миграции см. в разделе Использование «динамических» загрузок отношений без использования запроса ([Making use of “dynamic” relationship loads without using Query](https://docs.sqlalchemy.org/en/14/changelog/migration\_20.html#migration-20-dynamic-loaders)).
{% endhint %}

{% hint style="info" %}
Этот загрузчик в общем случае не совместим с расширением асинхронного ввода-вывода (asyncio) ([Asynchronous I/O (asyncio)](https://docs.sqlalchemy.org/en/14/orm/extensions/asyncio.html)). Его можно использовать с некоторыми ограничениями, как указано в динамических рекомендациях Asyncio ([Asyncio dynamic guidelines](https://docs.sqlalchemy.org/en/14/orm/extensions/asyncio.html#dynamic-asyncio)).
{% endhint %}

Отношение [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship), которое соответствует большой коллекции, можно настроить так, чтобы при доступе оно возвращало устаревший объект [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query), что позволяет фильтровать отношение по критериям. Класс представляет собой специальный класс [AppenderQuery](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.AppenderQuery), возвращаемый вместо коллекции при доступе. Критерий фильтрации, а также ограничения и смещения могут применяться как явно, так и через срезы массива:

```python
class User(Base):
    __tablename__ = 'user'

    posts = relationship(Post, lazy="dynamic")

jack = session.query(User).get(id)

# фильтровать посты блога Jack
posts = jack.posts.filter(Post.headline=='this is a post')

# применять срезы массива
posts = jack.posts[5:20]
```

Динамическая связь поддерживает ограниченные операции записи с помощью методов **AppenderQuery.append()** и **AppenderQuery.remove()**:

```python
oldpost = jack.posts.filter(Post.headline=='old post').one()
jack.posts.remove(oldpost)

jack.posts.append(Post('new post'))
```

Поскольку сторона чтения динамической связи всегда запрашивает базу данных, изменения в базовой коллекции не будут видны до тех пор, пока данные не будут сброшены. Однако, пока для используемого сеанса [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) включена функция «**autoflush**», это будет происходить автоматически каждый раз, когда коллекция собирается отправить запрос.

Чтобы поместить динамическую связь в обратную ссылку, используйте функцию [backref()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.backref) в сочетании с `lazy='dynamic'`:

```python
class Post(Base):
    __table__ = posts_table

    user = relationship(User,
                backref=backref('posts', lazy='dynamic')
            )
```

Обратите внимание, что параметры нетерпеливой/ленивой загрузки в настоящее время нельзя использовать в сочетании с динамическими отношениями.

| Название объекта                                                                                     | Описание                                                                 |
| ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| [AppenderQuery](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.AppenderQuery) | Динамический запрос, поддерживающий базовые операции хранения коллекций. |

### _class_ sqlalchemy.orm.AppenderQuery(_attr_, _state_)

Динамический запрос, поддерживающий базовые операции хранения коллекций.

{% hint style="info" %}
**Сигнатура класса**:

class [`sqlalchemy.orm.AppenderQuery`](https://docs.sqlalchemy.org/en/14/orm/collections.html#sqlalchemy.orm.AppenderQuery) (`sqlalchemy.orm.dynamic.AppenderMixin`, [`sqlalchemy.orm.Query`](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query))
{% endhint %}

{% hint style="info" %}
Функция [dynamic\_loader()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.dynamic\_loader) по сути такая же, как и функция [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) с указанным аргументом `lazy='dynamic'`.
{% endhint %}

{% hint style="danger" %}
«Динамический» загрузчик применяется **только к коллекциям**. Недопустимо использовать «динамические» загрузчики с отношениями «многие к одному», «один к одному» или `«uselist=False»`. В этих случаях более новые версии SQLAlchemy выдают предупреждения или исключения.
{% endhint %}

## Настройка Noload, RaiseLoad

Отношение «**noload**» никогда не загружается из базы данных, даже при доступе. Он настраивается с помощью `lazy='noload'`:

```python
class MyClass(Base):
    __tablename__ = 'some_table'

    children = relationship(MyOtherClass, lazy='noload')
```

Вышеупомянутая дочерняя коллекция **children** полностью доступна для записи, и изменения в ней будут сохраняться в базе данных, а также будут локально доступны для чтения в момент их добавления. Однако, когда экземпляры **MyClass** только что загружаются из базы данных, коллекция **children** остается пустой. Стратегия **noload** также доступна на основе параметра запроса с использованием параметра загрузчика [noload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.noload).

В качестве альтернативы, загруженное отношение «**raise**» вызовет [InvalidRequestError](https://docs.sqlalchemy.org/en/14/core/exceptions.html#sqlalchemy.exc.InvalidRequestError), где атрибут обычно выдает ленивую загрузку:

```python
class MyClass(Base):
    __tablename__ = 'some_table'

    children = relationship(MyOtherClass, lazy='raise')
```

Выше при доступе к атрибутам коллекции **children** возникнет исключение, если она не была предварительно загружена. Это включает в себя доступ для чтения, но для коллекций также повлияет на доступ для записи, поскольку коллекции не могут быть изменены без их предварительной загрузки. Смысл этого в том, чтобы убедиться, что приложение не выдает никаких неожиданных ленивых загрузок в определенном контексте. Вместо того, чтобы читать журналы SQL, чтобы определить, что все необходимые атрибуты были загружены с готовностью, стратегия «**raise**» вызовет немедленное поднятие исключения для незагруженных атрибутов при доступе к ним. Стратегия **raise** также доступна на основе параметра запроса с использованием параметра загрузчика [raiseload()](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#sqlalchemy.orm.raiseload).

{% hint style="warning" %}
**Новое в версии 1.1**: добавлена стратегия загрузчика «**raise**».
{% endhint %}

{% hint style="info" %}
Смотри также:

[Preventing unwanted lazy loads using raiseload](https://docs.sqlalchemy.org/en/14/orm/loading\_relationships.html#prevent-lazy-with-raiseload)
{% endhint %}

## Использование пассивных удалений

См. Использование каскада внешнего ключа ON DELETE с отношениями ORM для этого раздела ([Using foreign key ON DELETE cascade with ORM relationships](https://docs.sqlalchemy.org/en/14/orm/cascades.html#passive-deletes)).
