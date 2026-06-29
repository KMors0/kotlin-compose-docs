# Полное руководство по Kotlin, Compose Multiplatform и Material Design 3

> Комплексная документация по разработке кроссплатформенных приложений с использованием Kotlin, Compose Multiplatform и Material Design 3.

---

## 📑 Содержание

### 🟢 Для начинающих

| # | Раздел | Описание |
|---|--------|----------|
| ⭐ | [Словарь программиста: всё для начинающего](00-beginners-guide.md) | Классы, коллекции, асинхронность, null safety, архитектура, сетевые термины — простым языком с аналогиями |
| ⭐ | [Как собирать интерфейс в Compose](00-compose-layout.md) | Row, Column, Box, Modifier, padding, лейаут для тех, кто пришёл из Flutter |

### Введение

| # | Раздел | Описание |
|---|--------|----------|
| 00 | [Введение: Схождение трёх технологий](00-introduction.md) | Почему этот стек, обзор технологий, как пользоваться документацией |

### 🟢 Основы

| # | Раздел | Описание |
|---|--------|----------|
| 1 | [Основы Kotlin](01-kotlin-basics.md) | Синтаксис, типы, коллекции, null safety |
| 2 | [Compose Multiplatform](02-compose-multiplatform.md) | Установка, декларативный UI, состояние, платформенная логика |
| 3 | [Material Design 3](03-material-design-3.md) | Темы, компоненты, адаптивные UI, кастомизация |

### 🟡 Практика

| # | Раздел | Описание |
|---|--------|----------|
| 4 | [Практический пример: Список задач](04-practical-example.md) | Полное приложение от модели до запуска |
| 5 | [Работа с данными и хранилищами](05-data-storage.md) | SQLDelight, DataStore, синхронизация |
| 6 | [Интеграция с API и сетевые запросы](06-networking-api.md) | Ktor, JSON, обработка ошибок, аутентификация |

### 🟡 Расширенные темы

| # | Раздел | Описание |
|---|--------|----------|
| 7 | [Локализация и интернационализация](07-localization.md) | Мультиязычность, kotlin-i18n |
| 8 | [Расширенные возможности навигации](08-navigation.md) | Мультиплатформенная навигация, deep links |
| 9 | [Тестирование](09-testing.md) | Юнит-тесты, UI-тесты, E2E, покрытие |
| 10 | [Производительность и оптимизация](10-performance.md) | Рекомпоновки, кэширование, профилирование |
| 11 | [Платформенная интеграция](11-platform-integration.md) | Android, iOS, Web, Desktop |

### 🟡 Инфраструктура и качество

| # | Раздел | Описание |
|---|--------|----------|
| 12 | [Безопасность](12-security.md) | Хранение секретов, шифрование, защита от уязвимостей |
| 13 | [DevOps и CI/CD](13-devops-cicd.md) | Сборка, деплой, автоматизация релизов |
| 14 | [Архитектурные паттерны](14-architecture.md) | Clean Architecture, MVI, Dependency Injection |
| 15 | [Отладка и устранение ошибок](15-debugging.md) | Типичные проблемы, логирование, отладка на iOS |

### 🟡 Дополнительно

| # | Раздел | Описание |
|---|--------|----------|
| 16 | [Примеры реальных приложений](16-real-examples.md) | Погодник, соцсеть, калькулятор |
| 17 | [Обновления экосистемы](17-ecosystem-updates.md) | Дорожная карта, MD3 vs MD2, тренды |
| 18 | [Стиль кода и документация](18-code-style.md) | Kotlin Code Style, Dokka, именование |
| 19 | [Ресурсы для углубления](19-resources.md) | Книги, курсы, блоги, видео |

### 🔴 Глубокие погружения (In-Depth Research)

| # | Раздел | Описание |
|---|--------|----------|
| 20 | [Глубокое погружение в Kotlin](20-deep-dive-kotlin.md) | Система типов, корутины, K2, KMP-архитектура |
| 21 | [Глубокое погружение в Compose Multiplatform](21-deep-dive-compose.md) | Декларативная парадигма, матрица платформ, навигация |
| 22 | [Material Design 3: глубокое погружение](22-deep-dive-md3.md) | Дизайн-токены, M3 Expressive, адаптивность |
| 23 | [Интеграция и лучшие практики](23-integration-best-practices.md) | Настройка проекта, кроссплатформенное темирование |
| 24 | [Ограничения и контраргументы](24-limitations.md) | iOS-производительность, зрелость Web, экосистема |
| 25 | [Конкурентный ландшафт](25-competitive-landscape.md) | KMP vs Flutter vs React Native |
| 26 | [Заключение и перспективы](26-conclusion.md) | Ключевые выводы, рекомендации, дорожная карта |
| 27 | [Источники и литература](27-sources.md) | 28 проверенных источников |

### 📚 Справочник Kotlin (metanit.com)

Подробный учебник по языку Kotlin от metanit.com — 75 уроков с примерами кода, от переменных до корутин и Flow API.

👉 **[Открыть справочник metanit →](metanit/README.md)**

| Глава | Уроков | Темы |
|-------|--------|------|
| [1. Введение](metanit/01-intro/01-first-program.md) | 1 | Первая программа, настройка IDE |
| [2. Основы](metanit/02-basics/01-variables.md) | 10 | Переменные, типы, условия, циклы, массивы |
| [3. Функциональное программирование](metanit/03-functional/01-vararg.md) | 9 | Лямбды, замыкания, функции высшего порядка |
| [4. ООП](metanit/04-oop/01-constructors.md) | 14 | Классы, наследование, интерфейсы, data-классы |
| [5. Доп. возможности](metanit/05-advanced-oop/01-null-safety.md) | 7 | Null safety, scope-функции, перегрузка операторов |
| [6. Обобщения](metanit/06-generics/01-generic-classes-functions.md) | 2 | Generic-классы, ограничения |
| [7. Коллекции](metanit/07-collections/01-list.md) | 15 | List, Set, Map, Sequence, фильтрация, сортировка |
| [8. Корутины](metanit/08-coroutines/01-coroutine-scope.md) | 6 | launch, async, каналы, диспетчеры |
| [9. Асинхронные потоки](metanit/09-flows/01-creating-flow.md) | 8 | Flow API, map, filter, reduce, combine |
| [10. DSL](metanit/10-dsl/01-extension-receivers.md) | 1 | Функции расширения с получателем, DSL-билдеры |

### 📖 Официальная документация Kotlin (kotlinlang.ru)

Русскоязычный перевод официальной документации — 80+ статей по всем аспектам языка, от базового синтаксиса до корутин и KMP.

👉 **[Открыть документацию kotlinlang.ru →](kotlinlang/README.md)**

| Раздел | Статей | Темы |
|--------|--------|------|
| [Введение](kotlinlang/01-intro/basic-syntax.md) | 3 | Базовый синтаксис, идиомы, code style |
| [Типы](kotlinlang/02-basics-types/basic-types.md) | 2 | Базовые типы, приведение типов |
| [Управление потоком](kotlinlang/03-control-flow/control-flow.md) | 4 | if/when/for, исключения, пакеты |
| [Классы и объекты](kotlinlang/04-classes-objects/classes.md) | 17 | ООП: классы, наследование, интерфейсы, generics, делегирование |
| [Функции и лямбды](kotlinlang/05-functions-lambdas/functions.md) | 13 | Функции, лямбды, null safety, scope-функции, рефлексия |
| [Строители](kotlinlang/06-builders/type-safe-builders.md) | 2 | DSL, type-safe builders |
| [Мультиплатформа](kotlinlang/07-multiplatform/multiplatform.md) | 7 | KMP: мобильные, десктоп, зависимости |
| [Платформы](kotlinlang/08-platforms/comparison-to-java.md) | 8 | JVM, JS, Native, интероп |
| [Коллекции](kotlinlang/09-collections/collections-overview.md) | 18 | List/Set/Map, Sequence, все операции |
| [Корутины](kotlinlang/10-coroutines/coroutines-guide.md) | 5 | Руководство, basics, cancellation, dispatchers |
| [Справочник](kotlinlang/11-reference/keyword-reference.md) | 2 | Ключевые слова, грамматика |
| [Инструменты](kotlinlang/12-tools/gradle.md) | 6 | Gradle, Maven, KAPT, Dokka |
| [Обзор Kotlin](kotlinlang/13-overview/data-science-overview.md) | 2 | Data Science, спортивное программирование |
| [FAQ](kotlinlang/14-other/faq.md) | 1 | Часто задаваемые вопросы |

---

## 🚀 Быстрый старт

1. 🟢 **Только начинаете?** Начните с [Словаря программиста](00-beginners-guide.md) — все термины объяснены простым языком.
2. Прочитайте [Введение](00-introduction.md), чтобы понять, почему этот стек актуален.
3. Перейдите к [Основам Kotlin](01-kotlin-basics.md) — синтаксис, типы, коллекции.
4. Изучите [Compose Multiplatform](02-compose-multiplatform.md) для UI-фреймворка.
5. Освойте [Material Design 3](03-material-design-3.md) для создания красивых интерфейсов.
6. Соберите первое приложение по [Практическому примеру](04-practical-example.md).
7. Для глубокого понимания — изучите разделы 20–27.

## 🛠 Стек технологий

- **Kotlin** — основной язык программирования
- **Compose Multiplatform** — декларативный UI-фреймворк от JetBrains
- **Material Design 3** — дизайн-система от Google
- **SQLDelight** — кроссплатформенная ORM
- **Ktor** — HTTP-клиент для сетевых запросов
- **Koin / Dagger-Hilt** — Dependency Injection

## 📊 Уровни сложности

| Уровень | Разделы | Для кого |
|---------|---------|----------|
| 🟢 Начинающий | Словарь, 00–04, 19 | Те, кто начинает с Kotlin и Compose |
| 🟡 Продвинутый | 05–18 | Разработчики с опытом Kotlin |
| 🔴 Экспертный | 20–27 | Архитекторы, техлиды, принимающие решения |
