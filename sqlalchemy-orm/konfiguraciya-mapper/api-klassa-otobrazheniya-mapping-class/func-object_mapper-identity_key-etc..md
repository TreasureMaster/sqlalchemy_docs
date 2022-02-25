# func object\_mapper(), identity\_key(), etc.

## _function_ sqlalchemy.orm.object\_mapper(_instance_)

Учитывая объект, вернуть основной Mapper, связанный с экземпляром объекта.

Вызывает [sqlalchemy.orm.exc.UnmappedInstanceError](https://docs.sqlalchemy.org/en/14/orm/exceptions.html#sqlalchemy.orm.exc.UnmappedInstanceError), если сопоставление не настроено.

Эта функция доступна через систему контроля как:

```python
inspect(instance).mapper
```

Использование системы проверки вызовет [sqlalchemy.exc.NoInspectionAvailable](https://docs.sqlalchemy.org/en/14/core/exceptions.html#sqlalchemy.exc.NoInspectionAvailable), если экземпляр не является частью сопоставления.

## _function_ sqlalchemy.orm.class\_mapper(_class\__, _configure=True_)

Учитывая класс, вернуть основной Mapper, связанный с ключом.

Вызывает [UnmappedClassError](https://docs.sqlalchemy.org/en/14/orm/exceptions.html#sqlalchemy.orm.exc.UnmappedClassError), если для данного класса не настроено сопоставление, или [ArgumentError](https://docs.sqlalchemy.org/en/14/core/exceptions.html#sqlalchemy.exc.ArgumentError), если передан объект, не относящийся к классу.

Эквивалентная функциональность доступна через функцию [inspect()](https://docs.sqlalchemy.org/en/14/core/inspection.html#sqlalchemy.inspect) как:

```python
inspect(some_mapped_class)
```

Использование системы проверки вызовет [sqlalchemy.exc.NoInspectionAvailable](https://docs.sqlalchemy.org/en/14/core/exceptions.html#sqlalchemy.exc.NoInspectionAvailable), если класс не сопоставлен.

## _function_ sqlalchemy.orm.configure\_mappers()

Инициализирует отношения между всеми модулями отображения, которые были созданы на данный момент во всех коллекциях реестра [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl).

Этап конфигурации используется для согласования и инициализации взаимосвязей [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship) между отображаемыми классами, а также для вызова событий конфигурации, таких как [MapperEvents.before\_configured()](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.MapperEvents.before\_configured) и [MapperEvents.after\_configured()](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.MapperEvents.after\_configured), которые могут использоваться расширениями ORM или хуками пользовательских расширений.

Конфигурация mapper обычно вызывается автоматически при первом использовании сопоставлений из определенного реестра [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl), а также всякий раз, когда используются сопоставления и создаются дополнительные, еще не настроенные преобразователи. Однако процесс автоматической настройки является локальным только для [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl), включающего целевой преобразователь и любые связанные объекты [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl), от которых он может зависеть; это эквивалентно вызову метода [registry.configure()](class-registry.md#method-sqlalchemy.orm.registry.configure-cascade-false) для определенного реестра.

Напротив, функция **configure\_mappers()** вызывает процесс конфигурации для всех объектов [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl), существующих в памяти, и может быть полезна для сценариев, в которых используется множество отдельных объектов [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl), которые, тем не менее, взаимосвязаны.

{% hint style="warning" %}
**Изменено в версии 1.4**: Начиная с SQLAlchemy 1.4.0b2, эта функция работает отдельно для каждого [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl), определяя местонахождение всех имеющихся объектов реестра и вызывая для каждого из них метод [registry.configure()](class-registry.md#method-sqlalchemy.orm.registry.configure-cascade-false). Метод **registry.configure()** может быть предпочтительнее, чтобы ограничить конфигурацию преобразователей теми, которые являются локальными для определенного реестра [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl) и/или декларативного базового класса.
{% endhint %}

Точки, в которых вызывается автоматическая конфигурация, включают в себя создание экземпляра сопоставленного класса в экземпляре, а также когда запросы ORM выдаются с использованием [Session.query()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.query) или [Session.execute()](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.execute) с оператором с поддержкой ORM.

Процесс настройки mapper, независимо от того, вызывается он с помощью **configure\_mappers()** или из [registry.configure()](class-registry.md#method-sqlalchemy.orm.registry.configure-cascade-false), предоставляет несколько обработчиков событий, которые можно использовать для расширения шага настройки mapper. К таким хукам относятся:

* [MapperEvents.before\_configure()](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.MapperEvents.before\_configured) — вызывается один раз перед тем, как **configure\_mappers()** или [registry.configure()](class-registry.md#method-sqlalchemy.orm.registry.configure-cascade-false) выполняют какую-либо работу; это можно использовать для установки дополнительных опций, свойств или связанных сопоставлений перед продолжением операции.
* [MapperEvents.mapper\_configured()](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.MapperEvents.mapper\_configured) — вызывается, когда каждый отдельный Mapper настраивается в процессе; будет включать все состояние преобразователя, за исключением обратных ссылок, установленных другими преобразователями, которые еще предстоит настроить.
* [MapperEvents.after\_configure()](https://docs.sqlalchemy.org/en/14/orm/events.html#sqlalchemy.orm.MapperEvents.after\_configured) — вызывается один раз после завершения [configure\_mappers()](func-object\_mapper-identity\_key-etc..md#function-sqlalchemy.orm.configure\_mappers) или [registry.configure()](class-registry.md#method-sqlalchemy.orm.registry.configure-cascade-false); на этом этапе все объекты Mapper, подпадающие под действие операции конфигурации, будут полностью настроены. Обратите внимание, что вызывающее приложение может по-прежнему иметь другие сопоставления, которые еще не были созданы, например, если они находятся в модулях, которые еще не импортированы, а также могут иметь сопоставления, которые еще предстоит настроить, если они находятся в других коллекциях [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl), не являющихся частью текущей области конфигурации.

## _function_ sqlalchemy.orm.clear\_mappers()

Удаляет все mappers из всех классов.

{% hint style="warning" %}
**Изменено в версии 1.4**: теперь эта функция находит все объекты [registry](class-registry.md#class-sqlalchemy.orm.registry-metadata-none-class\_registry-none-constructor-less-than-function-\_decl) и вызывает метод [registry.dispose()](class-registry.md#method-sqlalchemy.orm.registry.dispose-cascade-false) каждого из них.
{% endhint %}

Эта функция удаляет все инструменты из классов и уничтожает связанные с ними преобразователи mappers. После вызова классы не сопоставляются, и позже их можно повторно сопоставить с новыми преобразователями.

**clear\_mappers()** не предназначен для обычного использования, так как его буквально нельзя использовать за пределами очень специфических сценариев тестирования. Обычно преобразователи являются постоянными структурными компонентами определяемых пользователем классов и никогда не удаляются независимо от их класса. Если сопоставленный класс сам является сборщиком мусора, его сопоставитель также автоматически удаляется. Таким образом, **clear\_mappers()** предназначен только для использования в тестовых наборах, которые повторно используют одни и те же классы с разными сопоставлениями, что само по себе является чрезвычайно редким вариантом использования - единственный такой вариант использования на самом деле - это собственный тестовый набор SQLAlchemy и, возможно, тест наборы других библиотек расширения ORM, которые предназначены для тестирования различных комбинаций построения преобразователя на фиксированном наборе классов.

## _function_ sqlalchemy.orm.util.identity\_key(_\*args_, _\*\*kwargs_)

Создайте кортежи «identity key», которые используются в качестве ключей в словаре [Session.identity\_map](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session.identity\_map).

Эта функция имеет несколько стилей вызова:

### identity\_key(class, ident, identity\_token=token)

Эта форма получает сопоставленный класс и скаляр или кортеж первичного ключа в качестве аргумента. Например:

```python
>>> identity_key(MyClass, (1, 2))
(<class '__main__.MyClass'>, (1, 2), None)
```

#### Параметры:

* **class** - сопоставленный класс (должен быть позиционным аргументом)
* **ident** - первичный ключ, может быть скалярным аргументом или аргументом кортежа.
* **identity\_token** - необязательный токен идентификации

{% hint style="warning" %}
**Новое в версии 1.2**: добавлен идентификатор **identity\_token**
{% endhint %}

### identity\_key(instance=instance)

Эта форма создаст идентификационный ключ для данного экземпляра. Экземпляр не обязательно должен быть постоянным, только его атрибуты первичного ключа должны быть заполнены (иначе ключ будет содержать `None` для этих отсутствующих значений).

Например:

```python
>>> instance = MyClass(1, 2)
>>> identity_key(instance=instance)
(<class '__main__.MyClass'>, (1, 2), None)
```

В этой форме данный экземпляр в конечном итоге запускается через [Mapper.identity\_key\_from\_instance()](https://docs.sqlalchemy.org/en/14/orm/mapping\_api.html#sqlalchemy.orm.Mapper.identity\_key\_from\_instance), что приведет к выполнению проверки базы данных для соответствующей строки, если срок действия объекта истек.

#### Параметры:

* **instance** - экземпляр объекта (должен быть указан как ключевое слово **arg**)

### identity\_key(class, row=row, identity\_token=token)

Эта форма похожа на форму класса/кортежа, за исключением того, что строка результата базы данных передается как объект [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row).

Например:

```python
>>> row = engine.execute(\
    text("select * from table where a=1 and b=2")\
    ).first()
>>> identity_key(MyClass, row=row)
(<class '__main__.MyClass'>, (1, 2), None)
```

#### Параметры:

* **class** - сопоставленный класс (должен быть позиционным аргументом)
* **row** - Строка [Row](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.Row), возвращаемая [CursorResult](https://docs.sqlalchemy.org/en/14/core/connections.html#sqlalchemy.engine.CursorResult) (должна быть указана как **keyword arg**)
* **identity\_token** - необязательный токен идентификации

{% hint style="warning" %}
**Новое в версии 1.2**: добавлен идентификатор **identity\_token**
{% endhint %}

## _function_ sqlalchemy.orm.polymorphic\_union(_table\_map_, _typecolname_, _aliasname='p\_union'_, _cast\_nulls=True_)

Создает оператор **UNION**, используемый полиморфным преобразователем.

См. [Наследование конкретных таблиц](../otobrazhenie-klassov-v-ierarkhii-nasledovaniya/nasledovanie-konkretnoi-tablicy.md#nasledovanie-konkretnoi-tablicy) для примера того, как это используется.

#### Параметры:

* **table\_map** - сопоставление полиморфных идентификаторов с объектами [Table](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table).
* **typecolname** - строковое имя столбца «дискриминатора», который будет получен из запроса, производящего полиморфную идентичность для каждой строки. Если `None`, полиморфный дискриминатор не создается.
* **aliasname** - имя сгенерированной конструкции [alias()](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.alias).
* **cast\_nulls** - если `True`, несуществующие столбцы, которые представлены как **NULL**, будут переданы в **CAST**. Это устаревшее поведение, которое проблематично для некоторых бэкендов, таких как **Oracle**, и в этом случае для него можно установить значение `False`.
