# Установление соединения — Engine

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Эта страница является частью [учебника по SQLAlchemy 1.4/2.0](./).

Предыдущий: [Учебник по SQLAlchemy 1.4 / 2.0](obzor-uchebnika.md) | Далее: [Работа с транзакциями и DBAPI](rabota-s-tranzakciyami-i-dbapi.md)
{% endhint %}

Начало любого приложения SQLAlchemy — это объект, называемый [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine). Этот объект действует как центральный источник соединений с конкретной базой данных, предоставляя как фабрику, так и пространство хранения, называемое [пулом соединений](https://docs.sqlalchemy.org/en/14/core/pooling.html) для этих соединений с базой данных. Механизм обычно представляет собой глобальный объект, созданный только один раз для определенного сервера базы данных и настроенный с использованием строки URL, которая описывает, как он должен подключаться к хосту базы данных или серверной части.

В этом руководстве мы будем использовать базу данных **SQLite** только в памяти. Это простой способ протестировать вещи, не требуя реальной уже существующей базы данных. [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine) создается с помощью [create\_engine()](https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create\_engine), указав для флага [create\_engine.future](https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create\_engine.params.future) значение `True`, чтобы мы в полной мере использовали использование [стиля 2.0](https://docs.sqlalchemy.org/en/14/glossary.html#term-2.0-style):

```python
>>> from sqlalchemy import create_engine
>>> engine = create_engine("sqlite+pysqlite:///:memory:", echo=True, future=True)
```

Основным аргументом [create\_engine](https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create\_engine) является строковый URL-адрес, переданный выше как строка `«sqlite+pysqlite:///:memory:»`. Эта строка указывает [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine) на три важных факта:

1. С какой базой данных мы общаемся? Это часть **sqlite** выше, которая ссылается в SQLAlchemy на объект, известный как [диалект](https://docs.sqlalchemy.org/en/14/glossary.html#term-dialect).
2. Какой [DBAPI](https://docs.sqlalchemy.org/en/14/glossary.html#term-DBAPI) мы используем? Python [DBAPI](https://docs.sqlalchemy.org/en/14/glossary.html#term-DBAPI) — это сторонний драйвер, который SQLAlchemy использует для взаимодействия с конкретной базой данных. В данном случае мы используем имя **pysqlite**, которое в современном языке Python является интерфейсом стандартной библиотеки [sqlite3](https://docs.python.org/library/sqlite3.html) для SQLite. Если этот параметр опущен, SQLAlchemy будет использовать [DBAPI](https://docs.sqlalchemy.org/en/14/glossary.html#term-DBAPI) по умолчанию для конкретной выбранной базы данных.
3. Как мы находим базу данных? В этом случае наш URL-адрес включает фразу `/:memory:`, которая указывает модулю **sqlite3**, что мы будем использовать базу данных **только в памяти**. Этот тип базы данных идеально подходит для экспериментов, поскольку он не требует сервера и не требует создания новых файлов.

{% hint style="info" %}
Ленивое подключение

[Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine), впервые возвращенный функцией [create\_engine()](https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create\_engine), еще не пытался подключиться к базе данных; это происходит только в первый раз, когда его просят выполнить задачу для базы данных. Это шаблон проектирования программного обеспечения, известный как [ленивая инициализация](https://docs.sqlalchemy.org/en/14/glossary.html#term-lazy-initialization).
{% endhint %}

Мы также указали параметр [create\_engine.echo](https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create\_engine.params.echo), который даст указание [Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine) регистрировать все SQL, которые он выдает, в логгер Python, который будет записывать в стандартный вывод. Этот флаг представляет собой сокращенный способ более формальной [настройки ведения журнала Python](https://docs.sqlalchemy.org/en/14/core/engines.html#dbengine-logging) и полезен для экспериментов со сценариями. Многие примеры SQL будут включать эти выходные данные журнала SQL под ссылкой `[SQL]`, при нажатии на которую будет показано полное взаимодействие SQL.

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел руководства: [Работа с транзакциями и DBAPI](rabota-s-tranzakciyami-i-dbapi.md)
{% endhint %}
