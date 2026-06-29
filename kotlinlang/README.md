# Официальная документация Kotlin (kotlinlang.ru)

> Русскоязычный перевод официальной документации Kotlin с [kotlinlang.ru](https://kotlinlang.ru) — наиболее полное авторитетное руководство по языку от сообщества.

---

## 📑 Содержание

### [Введение](01-intro/basic-syntax.md)
Базовый синтаксис, идиомы, соглашения по коду.

| Статья | Описание |
|--------|----------|
| [Базовый синтаксис](01-intro/basic-syntax.md) | Первые шаги: функции, переменные, классы |
| [Идиомы](01-intro/idioms.md) | Идиоматичные конструкции Kotlin |
| [Соглашения по коду](01-intro/coding-conventions.md) | Официальный стиль кода от JetBrains |

### [Типы](02-basics-types/basic-types.md)
Система типов Kotlin, преобразования.

| Статья | Описание |
|--------|----------|
| [Базовые типы](02-basics-types/basic-types.md) | Числа, строки, массивы, беззнаковые типы |
| [Приведение типов](02-basics-types/typecasts.md) | is, as, smart casts |

### [Управление потоком](03-control-flow/control-flow.md)
Условия, циклы, исключения, пакеты.

| Статья | Описание |
|--------|----------|
| [Управление потоком](03-control-flow/control-flow.md) | if, when, for, while |
| [Возвраты и переходы](03-control-flow/returns.md) | return, break, continue, метки |
| [Исключения](03-control-flow/exceptions.md) | try/catch/finally, пользовательские исключения |
| [Пакеты](03-control-flow/packages.md) | package, import, as |

### [Классы и объекты](04-classes-objects/classes.md)
Полное руководство по ООП в Kotlin.

| Статья | Описание |
|--------|----------|
| [Классы](04-classes-objects/classes.md) | Объявление, конструкторы, члены класса |
| [Наследование](04-classes-objects/inheritance.md) | open, override, super |
| [Свойства](04-classes-objects/properties.md) | val/var, getters/setters, backing fields |
| [Интерфейсы](04-classes-objects/interfaces.md) | Методы, свойства, реализация по умолчанию |
| [Функциональные интерфейсы](04-classes-objects/fun-interfaces.md) | SAM-преобразования, fun interface |
| [Модификаторы видимости](04-classes-objects/visibility-modifiers.md) | public, private, protected, internal |
| [Функции расширения](04-classes-objects/extensions.md) | Расширение классов без наследования |
| [Data-классы](04-classes-objects/data-classes.md) | Автогенерация equals/hashCode/toString/copy |
| [Sealed-классы](04-classes-objects/sealed-classes.md) | Ограниченные иерархии |
| [Обобщения](04-classes-objects/generics.md) | variance, star projection, reified |
| [Вложенные классы](04-classes-objects/nested-classes.md) | nested, inner |
| [Enum-классы](04-classes-objects/enum-classes.md) | Перечисления с методами |
| [Inline-классы](04-classes-objects/inline-classes.md) | Типобезопасные обёртки без overhead |
| [Объекты](04-classes-objects/object-declarations.md) | object, companion object |
| [Делегирование](04-classes-objects/delegation.md) | Делегирование классов |
| [Делегированные свойства](04-classes-objects/delegated-properties.md) | by lazy, by observable, кастомные делегаты |
| [Псевдонимы типов](04-classes-objects/type-aliases.md) | type aliases для упрощения |

### [Функции и лямбды](05-functions-lambdas/functions.md)
Функции, лямбды, null safety, scope-функции.

| Статья | Описание |
|--------|----------|
| [Функции](05-functions-lambdas/functions.md) | Параметры, значения по умолчанию, vararg |
| [Лямбды](05-functions-lambdas/lambdas.md) | Высокоуровневые функции, it, замыкания |
| [Inline-функции](05-functions-lambdas/inline-functions.md) | inline, noinline, crossinline, reified |
| [Перегрузка операторов](05-functions-lambdas/operator-overloading.md) | operator fun, конвенции |
| [Null Safety](05-functions-lambdas/null-safety.md) | ?, ?., ?:, !!, let |
| [Равенство](05-functions-lambdas/equality.md) | ==, ===, structural vs referential |
| [Выражения this](05-functions-lambdas/this-expressions.md) | this@Label |
| [Асинхронное программирование](05-functions-lambdas/async-programming.md) | Обзор подходов |
| [Обзор корутин](05-functions-lambdas/coroutines-overview.md) | Введение в корутины |
| [Аннотации](05-functions-lambdas/annotations.md) | Аннотации, мета-аннотации |
| [Деструктурирующие объявления](05-functions-lambdas/destructuring-declarations.md) | componentN(), for, in |
| [Рефлексия](05-functions-lambdas/reflection.md) | KClass, KFunction, KProperty |
| [Scope-функции](05-functions-lambdas/scope-functions.md) | let, run, with, apply, also |

### [Строители (Builders)](06-builders/type-safe-builders.md)
Типобезопасные DSL и билдеры.

| Статья | Описание |
|--------|----------|
| [Типобезопасные строители](06-builders/type-safe-builders.md) | DSL на Kotlin, HTML builder |
| [Builder inference](06-builders/using-builders-with-builder-inference.md) | Вывод типов в строителях |

### [Мультиплатформенная разработка](07-multiplatform/multiplatform.md)
Kotlin Multiplatform: мобильные, десктоп, веб.

| Статья | Описание |
|--------|----------|
| [Обзор KMP](07-multiplatform/multiplatform.md) | Концепция мультиплатформенности |
| [Мобильная разработка: начало](07-multiplatform/mobile-getting-started.md) | KMP для iOS и Android |
| [Настройка мобильного проекта](07-multiplatform/mobile-setup.md) | Среда разработки |
| [Первое мобильное приложение](07-multiplatform/mobile-create-first-app.md) | Пошаговое руководство |
| [Начало работы (все платформы)](07-multiplatform/get-started.md) | KMP для всех платформ |
| [DSL-справочник](07-multiplatform/dsl-reference.md) | Gradle DSL для KMP |
| [Добавление зависимостей](07-multiplatform/add-dependencies.md) | Управление зависимостями |

### [Платформы](08-platforms/comparison-to-java.md)
JVM, JS, Native, интероперабельность.

| Статья | Описание |
|--------|----------|
| [Сравнение с Java](08-platforms/comparison-to-java.md) | Отличия от Java |
| [Java-интероп](08-platforms/java-interop.md) | Вызов Java из Kotlin |
| [Kotlin-Java интероп](08-platforms/java-to-kotlin-interop.md) | Вызов Kotlin из Java |
| [Dynamic тип](08-platforms/dynamic-type.md) | dynamic для Kotlin/JS |
| [Kotlin/JS](08-platforms/js-overview.md) | Разработка для веба |
| [Kotlin/Native](08-platforms/native-overview.md) | Нативная компиляция |
| [Android](08-platforms/android-overview.md) | Разработка под Android |
| [Серверная разработка](08-platforms/server-overview.md) | Backend на Kotlin |

### [Коллекции](09-collections/collections-overview.md)
Полное руководство по коллекциям Kotlin.

| Статья | Описание |
|--------|----------|
| [Обзор коллекций](09-collections/collections-overview.md) | List, Set, Map, изменяемость |
| [Создание коллекций](09-collections/constructing-collections.md) | Конструкторы, фабричные функции |
| [Итераторы](09-collections/iterators.md) | Iterator, ListIterator |
| [Диапазоны](09-collections/ranges.md) | IntRange, CharRange, прогрессии |
| [Последовательности](09-collections/sequences.md) | Ленивые вычисления, Sequence |
| [Операции: обзор](09-collections/collection-operations.md) | Категории операций |
| [Трансформации](09-collections/collection-transformations.md) | map, zip, associate, flatten |
| [Фильтрация](09-collections/collection-filtering.md) | filter, take, drop, slice |
| [Плюс и минус](09-collections/collection-plus-minus.md) | Объединение, вычитание |
| [Группировка](09-collections/collection-grouping.md) | groupBy, groupingBy |
| [Части коллекций](09-collections/collection-parts.md) | take, drop, chunked, windowed |
| [Получение элементов](09-collections/collection-elements.md) | get, find, first, last, random |
| [Сортировка](09-collections/collection-ordering.md) | sorted, sortedBy, compareBy |
| [Агрегация](09-collections/collection-aggregate.md) | reduce, fold, count, sum |
| [Изменение коллекций](09-collections/collection-write.md) | add, remove, update |
| [Операции List](09-collections/list-operations.md) | Специфичные для List |
| [Операции Set](09-collections/set-operations.md) | Специфичные для Set |
| [Операции Map](09-collections/map-operations.md) | Специфичные для Map |

### [Корутины (kotlinx.coroutines)](10-coroutines/coroutines-guide.md)
Официальное руководство по корутинам.

| Статья | Описание |
|--------|----------|
| [Полное руководство](10-coroutines/coroutines-guide.md) | Гайд от начала до продвинутых тем |
| [Основы](10-coroutines/coroutines-basics.md) | first, launch, delay, suspend |
| [Отмена и таймауты](10-coroutines/cancellation-and-timeouts.md) | cancel(), withTimeout() |
| [Композиция suspend-функций](10-coroutines/composing-suspending-functions.md) | async, launch, последовательный/параллельный запуск |
| [Контекст и диспетчеры](10-coroutines/coroutine-context-and-dispatchers.md) | Dispatchers, withContext, Job |

### [Справочник](11-reference/keyword-reference.md)
Ключевые слова, грамматика.

| Статья | Описание |
|--------|----------|
| [Ключевые слова](11-reference/keyword-reference.md) | Все ключевые слова Kotlin |
| [Грамматика](11-reference/grammar.md) | Формальная грамматика Kotlin |

### [Инструменты](12-tools/gradle.md)
Сборка, плагины, документация.

| Статья | Описание |
|--------|----------|
| [Gradle](12-tools/gradle.md) | Сборка с Gradle |
| [Maven](12-tools/maven.md) | Сборка с Maven |
| [Ant](12-tools/ant.md) | Сборка с Ant |
| [All-open плагин](12-tools/all-open-plugin.md) | Авто-open для классов |
| [KAPT](12-tools/kapt.md) | Annotation processing |
| [Kotlin Doc](12-tools/kotlin-doc.md) | Генерация документации |

### [Обзор Kotlin](13-overview/data-science-overview.md)
Области применения Kotlin.

| Статья | Описание |
|--------|----------|
| [Data Science](13-overview/data-science-overview.md) | Kotlin для Data Science |
| [Спортивное программирование](13-overview/competitive-programming.md) | Kotlin для контестов |

### [Прочее](14-other/faq.md)
FAQ и полезное.

| Статья | Описание |
|--------|----------|
| [FAQ](14-other/faq.md) | Часто задаваемые вопросы |

---

## 🔗 Связь с основной документацией

| Раздел основной документации | Соответствующий раздел kotlinlang.ru |
|------------------------------|--------------------------------------|
| [01. Основы Kotlin](../01-kotlin-basics.md) | Введение + Типы + Управление потоком |
| [20. Глубокое погружение в Kotlin](../20-deep-dive-kotlin.md) | Функции + Классы + Null Safety |
| [05. Работа с данными](../05-data-storage.md) | Коллекции |
| [02. Compose Multiplatform](../02-compose-multiplatform.md) | Мультиплатформенная разработка |

---

*Источник: [kotlinlang.ru](https://kotlinlang.ru) — русскоязычный перевод официальной документации Kotlin*
