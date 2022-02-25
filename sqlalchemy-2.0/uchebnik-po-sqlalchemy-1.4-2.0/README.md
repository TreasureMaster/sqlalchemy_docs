# Учебник по SQLAlchemy 1.4/2.0

{% hint style="info" %}
**Об этом документе**:

Новое учебное пособие по SQLAlchemy теперь интегрировано между Core и ORM и служит унифицированным введением в SQLAlchemy в целом. В новом [стиле работы 2.0](https://docs.sqlalchemy.org/en/14/glossary.html#term-2.0-style), полностью доступном в [версии 1.4](https://docs.sqlalchemy.org/en/14/changelog/migration\_14.html), ORM теперь использует запросы в стиле Core с конструкцией [select()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select), а семантика транзакций между соединениями Core и сеансами ORM эквивалентна. Обратите внимание на стили синей границы для каждого раздела, которые расскажут вам, насколько «ORM-подобна» конкретная тема!

Пользователи, которые уже знакомы с SQLAlchemy, и особенно те, кто хочет перенести существующие приложения для работы с SQLAlchemy 2.0 на переходном этапе версии 1.4, также должны ознакомиться с документом Миграция на SQLAlchemy 2.0 ([Migrating to SQLAlchemy 2.0](https://docs.sqlalchemy.org/en/14/changelog/migration\_20.html)).

Для новичка в этом документе **много** деталей, однако к концу он будет считаться **Алхимиком**.
{% endhint %}

SQLAlchemy представлен как два разных API, один над другим. Эти API известны как **Core** и **ORM**.

> **SQLAlchemy Core** является базовой архитектурой для SQLAlchemy как «набора инструментов для работы с базами данных». Библиотека предоставляет инструменты для управления подключением к базе данных, взаимодействия с запросами и результатами базы данных, а также для программного построения операторов SQL.
>
> Разделы, _**обведенные** серой **рамкой слева (как цитата)**_, будут обсуждать концепции, которые в основном **относятся только к Core**; при использовании ORM эти концепции все еще используются, но реже явно выражены в пользовательском коде.

{% hint style="success" %}
**SQLAlchemy ORM** основывается на ядре **Core**, предоставляя дополнительные возможности **реляционного сопоставления объектов**. ORM предоставляет дополнительный уровень конфигурации, позволяющий **сопоставлять** определяемые пользователем классы Python с таблицами базы данных и другими конструкциями, а также механизм сохранения объектов, известный как сеанс **Session**. Затем он расширяет язык выражений SQL базового уровня, позволяя составлять и вызывать SQL-запросы с точки зрения пользовательских объектов.

В разделах, _**обведенных **<mark style="color:green;">**зеленой**</mark>** рамкой слева**_, обсуждаются концепции, в основном предназначенные только для ORM. Пользователи **только Core** могут их пропустить.
{% endhint %}

{% hint style="warning" %}
В разделе, который _**обведен **<mark style="color:orange;">**оранжевой**</mark>** рамой слева**_, будет обсуждаться **концепция Core, которая также явно используется с ORM**.
{% endhint %}