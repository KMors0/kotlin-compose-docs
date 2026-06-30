# Полное руководство по Kotlin, Compose Multiplatform и Material Design 3

> Комплексная документация по разработке кроссплатформенных приложений с использованием Kotlin 2.4, Compose Multiplatform 1.11 и Material Design 3 Expressive. Документация актуализирована на **30 июня 2026**.

---

## 📑 Содержание

### 🟢 Для начинающих (с нуля)

| # | Раздел | Описание |
|---|--------|----------|
| ⭐ | [Программирование с нуля: концепции через Kotlin](00-programming-foundations.md) | Что такое функция, класс, объект, тип, цикл — концептуальное погружение для тех, кто никогда не программировал |
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
| 7 | [Локализация и интернационализация](07-localization.md) | Compose Resources, Lyricist, плюрализация, смена языка |
| 8 | [Расширенные возможности навигации](08-navigation.md) | navigation-compose, Decompose, Voyager, deep links |
| 9 | [Тестирование](09-testing.md) | kotlin-test, Mockative, Compose UI Test, Kover, E2E |
| 10 | [Производительность и оптимизация](10-performance.md) | Рекомпоновки, кэширование, профилирование, бенчмарки |
| 11 | [Платформенная интеграция](11-platform-integration.md) | Android, iOS, Web, Desktop, медиа, фоновые задачи |

### 🟡 Инфраструктура и качество

| # | Раздел | Описание |
|---|--------|----------|
| 12 | [Безопасность](12-security.md) | Хранение секретов, шифрование, защита от уязвимостей |
| 13 | [DevOps и CI/CD](13-devops-cicd.md) | GitHub Actions @v4, Codemagic, Bitrise, автоматизация релизов |
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

### 🔵 Практические руководства

| # | Раздел | Описание |
|---|--------|----------|
| 28 | [Доступность (Accessibility)](28-accessibility.md) | Semantics API, скринридеры, контраст, touch targets, тестирование a11y |
| 29 | [Анимации и Motion](29-animations-motion.md) | animate*AsState, AnimatedVisibility, spring-физика, M3 Expressive, жесты |
| 30 | [Миграция на KMP](30-migration-guide.md) | Пошаговый гайд: Android → KMP → iOS, таймлайн, чеклист |
| 31 | [Шпаргалка (Quick Reference)](31-quick-reference.md) | Структура проекта, Gradle-команды, зависимости, version matrix |
| 32 | [Управление состоянием: deep dive](32-state-management-deep-dive.md) | Snapshot, derivedStateOf, StateFlow, remember vs rememberSaveable, UDF |
| 33 | [expect/actual: паттерны и антипаттерны](33-expect-actual-patterns.md) | 5 production-паттернов, 4 антипаттерна, тестирование, отладка |
| 34 | [KMP Gotchas: подводные камни](34-kmp-gotchas.md) | New MM, freeze(), AtomicFU, retain cycles, serialization polymorphism |
| 35 | [Адаптивные лейауты и foldables](31-adaptive-layouts.md) | WindowSizeClass, NavigationBar/Rail/Drawer, ListDetailPaneScaffold |
| 36 | [Оптимизация размера приложения](37-app-size-optimization.md) | R8/ProGuard, shrinking iOS binaries, wasm tree-shaking, ресурсы |
| 37 | [P2P-сети и мессенджеры](38-p2p-networking.md) | QUIC, WebRTC, STUN/TURN/ICE, E2E-шифрование, сигналинг, синхронизация между устройствами, push-уведомления в P2P |

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

1. 🟢 **Никогда не программировали?** Начните с [Программирования с нуля](00-programming-foundations.md) — концептуальное объяснение: что такое функция, класс, переменная, цикл, через аналогии и примеры.
2. Если знаете основы программирования, но не знаете Kotlin — [Словарь программиста](00-beginners-guide.md) объясняет термины простым языком.
3. Прочитайте [Введение](00-introduction.md), чтобы понять, почему этот стек актуален.
4. Перейдите к [Основам Kotlin](01-kotlin-basics.md) — синтаксис, типы, коллекции.
5. Изучите [Compose Multiplatform](02-compose-multiplatform.md) для UI-фреймворка (настройка через [KMP Wizard](https://kmp.jetbrains.com/)).
6. Освойте [Material Design 3](03-material-design-3.md) для создания красивых интерфейсов.
7. Соберите первое приложение по [Практическому примеру](04-practical-example.md).
8. Для глубокого понимания — изучите разделы 20–27.

## 🛠 Стек технологий (актуальные версии на 30 июня 2026)

- **Kotlin 2.4.0** — основной язык программирования (2.4.10-RC, 2.4.20-Beta1 в EAP)
- **Compose Multiplatform 1.11.0** — декларативный UI-фреймворк от JetBrains
- **Material Design 3 Expressive** — дизайн-система от Google (compose-material3 1.4.x)
- **SQLDelight 2.1+** — кроссплатформенная ORM (groupId `app.cash.sqldelight`)
- **Ktor 3.4.0** — HTTP-клиент для сетевых запросов
- **kotlinx-coroutines 1.11.0** — асинхронное программирование
- **Koin 4.0+ / Dagger-Hilt** — Dependency Injection
- **AndroidX Navigation 2.9+** — type-safe навигация (работает на всех платформах с CMP 1.10+)

> **См. также:** [Обновления экосистемы](17-ecosystem-updates.md) — полная таблица версий и дат релизов.

## 📊 Уровни сложности

| Уровень | Разделы | Для кого |
|---------|---------|----------|
| 🟢 Начинающий (с нуля) | [Программирование с нуля](00-programming-foundations.md), Словарь, 00–04, 19 | Те, кто начинает с Kotlin и Compose |
| 🟡 Продвинутый | 05–18 | Разработчики с опытом Kotlin |
| 🔵 Практические | 28–37 | Разработчики, решающие конкретные задачи |
| 🔴 Экспертный | 20–27 | Архитекторы, техлиды, принимающие решения |
