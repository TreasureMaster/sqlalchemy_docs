# Обзор учебника

В учебнике обе концепции будут представлены в том порядке, в котором их следует изучать, сначала с подходом, в основном ориентированным на Core, а затем с переходом на более ориентированные на ORM концепции.

Основные разделы этого руководства следующие:

* [Установление соединения — Engine](ustanovlenie-soedineniya-engine.md) — все приложения SQLAlchemy начинаются с объекта [Engine](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Engine); здесь описано как его создать.
* [Работа с транзакциями и DBAPI](rabota-s-tranzakciyami-i-dbapi.md) — здесь представлены API использования [Engine](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Engine) и связанные с ним объекты [Connection](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Connection) и [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result). Этот контент ориентирован на Core, однако пользователи ORM захотят ознакомиться хотя бы с объектом [Result](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Result).
* [Работа с метаданными базы данных](rabota-s-metadannymi-bazy-dannykh.md) - SQL-абстракции SQLAlchemy, а также ORM полагаются на систему определения конструкций схемы базы данных как объектов Python. В этом разделе рассказывается, как это сделать с точки зрения Core и ORM.
* [Работа с данными](rabota-s-dannymi/) — здесь мы учимся создавать, выбирать, обновлять и удалять данные в базе данных. Так называемые операции [CRUD](https://docs.sqlalchemy.org/en/14/glossary.html#term-CRUD) здесь даны в терминах SQLAlchemy Core со ссылками на их аналоги ORM. Операция **SELECT**, которая подробно описана в разделе Выбор строк с помощью Core или ORM ([Selecting Rows with Core or ORM](https://docs.sqlalchemy.org/en/14/tutorial/data\_select.html#tutorial-selecting-data)), одинаково хорошо применима к Core и ORM.
* [Манипуляция данными с помощью ORM](manipulyacii-s-dannymi-s-pomoshyu-orm.md) - охватывает структуру сохраняемости ORM; в основном ORM-ориентированные способы вставки, обновления и удаления, а также способы обработки транзакций.
* [Работа со связанными объектами](rabota-so-svyazannymi-obektami.md) - знакомит с концепцией конструкции [relationship()](../../sqlalchemy-orm/konfiguraciya-relationship/relationship-api.md#function-sqlalchemy.orm.relationship-argument-secondary-none-primaryjoin-none-secondaryjoin-none-for) и дает краткий обзор того, как она используется, со ссылками на более подробную документацию.
* [Дополнительное чтение](dopolnitelnoe-chtenie.md) - в этом разделе перечислены основные разделы документации верхнего уровня, в которых полностью описаны концепции, представленные в этом руководстве.

## Проверка версии

Этот учебник написан с использованием системы под названием [doctest](https://docs.python.org/3/library/doctest.html). Все фрагменты кода, написанные с помощью **`>>>`**, на самом деле выполняются как часть набора тестов SQLAlchemy, и читателю предлагается работать с приведенными примерами кода в режиме реального времени с помощью собственного интерпретатора Python.

При запуске примеров рекомендуется, чтобы читатель выполнил быструю проверку, чтобы убедиться, что мы используем **версию 1.4** SQLAlchemy:

```python
>>> import sqlalchemy
>>> sqlalchemy.__version__  
1.4.0
```

## Заметка на будущее

В этом руководстве описывается новый API, выпущенный в SQLAlchemy 1.4, известный как [стиль 2.0](https://docs.sqlalchemy.org/en/14/glossary.html#term-2.0-style). Целью API в стиле 2.0 является обеспечение прямой совместимости с [SQLAlchemy 2.0](https://docs.sqlalchemy.org/en/14/changelog/migration\_20.html), который планируется как следующее поколение SQLAlchemy.

Чтобы предоставить полный API 2.0, будет использоваться новый флаг с именем **future**, который будет отображаться по мере того, как в руководстве описываются объекты [Engine](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Engine) и [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session). Эти флаги полностью включают режим совместимости с версией 2.0 и позволяют полностью выполнять код в руководстве. При использовании флага **future** с функцией [create\_engine()](https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create\_engine) возвращаемый объект является подклассом [sqlalchemy.engine.Engine](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Engine), описанным как [sqlalchemy.future.Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine). В этом руководстве будет использоваться [sqlalchemy.future.Engine](https://docs.sqlalchemy.org/en/14/core/future.html#sqlalchemy.future.Engine).

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел: [Установление соединения — Engine](ustanovlenie-soedineniya-engine.md)
{% endhint %}
