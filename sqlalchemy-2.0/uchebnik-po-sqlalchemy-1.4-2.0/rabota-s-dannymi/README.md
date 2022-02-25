# Работа с данными

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Эта страница является частью [учебника по SQLAlchemy 1.4/2.0](../).

Предыдущий: [Работа с метаданными базы данных](../rabota-s-metadannymi-bazy-dannykh.md) | Далее: [Вставка строк с Core](vstavka-strok-s-core.md)
{% endhint %}

В разделе «[Работа с транзакциями и DBAPI](../rabota-s-tranzakciyami-i-dbapi.md)» мы изучили основы того, как взаимодействовать с DBAPI Python и его транзакционным состоянием. Затем в разделе «[Работа с метаданными базы данных](../rabota-s-metadannymi-bazy-dannykh.md)» мы узнали, как представлять таблицы, столбцы и ограничения базы данных в SQLAlchemy, используя метаданные [Metadata](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.MetaData) и связанные объекты. В этом разделе мы объединим обе приведенные выше концепции для создания, выбора и управления данными в реляционной базе данных. Наше взаимодействие с базой данных _**всегда**_ осуществляется с точки зрения транзакции, даже если мы настроили драйвер базы данных на использование автоматической фиксации [autocommit](https://docs.sqlalchemy.org/en/14/core/connections.html#dbapi-autocommit) за кулисами.

Компоненты этого раздела следующие:

* [Вставка строк с Core](vstavka-strok-s-core.md) — чтобы получить некоторые данные в базу данных, мы вводим и демонстрируем конструкцию Core [Insert](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Insert). **INSERT** с точки зрения ORM описаны в следующем разделе [Манипуляции с данными с помощью ORM](../manipulyacii-s-dannymi-s-pomoshyu-orm.md).
* [Выбор строк с помощью Core или ORM](vybor-strok-s-pomoshyu-core-ili-orm.md) — в этом разделе будет подробно описана конструкция [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select), которая является наиболее часто используемым объектом в SQLAlchemy. Конструкция [Select](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select) генерирует операторы **SELECT** для приложений, ориентированных как на Core, так и на ORM, и оба варианта использования будут описаны здесь. Дополнительные варианты использования ORM также указаны в следующем разделе «[Использование отношений в запросах](../rabota-so-svyazannymi-obektami.md#ispolzovanie-otnoshenii-relationships-v-zaprosakh-query)», а также в Руководстве по созданию запросов ORM ([ORM Querying Guide](https://docs.sqlalchemy.org/en/14/orm/queryguide.html)).
* [Обновление и удаление строк с помощью Core](obnovlenie-i-udalenie-strok-s-pomoshyu-core.md) - завершая операции **INSERT** и **SELECT**ion данных, в этом разделе с точки зрения Core будет описано использование конструкций [Update](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Update) и [Delete](https://docs.sqlalchemy.org/en/14/core/dml.html#sqlalchemy.sql.expression.Delete). **UPDATE** и **DELETE**, специфичные для ORM, аналогично описаны в разделе «[Обработка данных с помощью ORM](../manipulyacii-s-dannymi-s-pomoshyu-orm.md)».

{% hint style="info" %}
**Учебник по SQLAlchemy 1.4/2.0**

Следующий раздел руководства: [Вставка строк с Core](vstavka-strok-s-core.md)
{% endhint %}
