# Ченджлог улучшений документации (GLM 5.1 → GLM 5.2)

**Дата апдейта:** 30 июня 2026  
**Версия-ориентир:** Kotlin 2.4.0, Compose Multiplatform 1.11.0, Material 3 Expressive, kotlinx-coroutines 1.11.0, Ktor 3.4.0, SQLDelight 2.1+  
**Метод:** активный веб-поиск для проверки актуальных версий и API

---

## Сводка изменений по главам

### Глава 17 — Обновления экосистемы (полная переработка)
**Было:** ~50 строк общих фраз без конкретных версий ("следите за обновлениями", "JetBrains регулярно выпускает обновления").  
**Стало:** ~180 строк с конкретными версиями, датами и changelog'ом.

**Ключевые изменения:**
- Добавлена точная таблица версий на 30 июня 2026: Kotlin 2.4.0 (stable) / 2.4.10-RC / 2.4.20-Beta1, CMP 1.11.0, AndroidX Compose 1.11.2, kotlinx-coroutines 1.11.0, Ktor 3.4.0, SQLDelight 2.1+.
- Добавлены даты: Kotlin 2.3.21 от 23 апреля 2026; kotlinx-coroutines 1.11.0 от 9 мая 2026.
- Добавлен раздел "Ключевые вехи 2026 года" с конкретными месяцами и фичами (Swift packages в Kotlin 2.4, Unified @Preview в CMP 1.10, coroutine stack trace recovery в 2.4.20-Beta1).
- Расширено сравнение MD3 vs M3 Expressive: добавлены LoadingIndicator, ContainedLoadingIndicator, SplitButton, FloatingActionButtonMenu с примерами кода.
- Добавлены тренды 2026: Swift export, KSP2, Navigation 3 на non-Android, Wasm как основной web-таргет.
- Добавлен практический чеклист обновления проекта с Kotlin 2.1 → 2.4.

### Глава 21 — Глубокое погружение в Compose Multiplatform (полная переработка)
**Было:** версия CMP 1.8.0 упомянута как "стабильная с мая 2025", Web (Kotlin/Wasm) — Beta.  
**Стало:** CMP 1.11.0, Web (Wasm) — Stable-ready, добавлены новые фичи 1.10–1.11.

**Ключевые изменения:**
- Обновлена матрица платформ: Web (Kotlin/Wasm) теперь Stable-ready с 1.11.0, Kotlin/JS помечен как устаревший.
- Добавлена новая секция "21.5. Новые возможности Compose Multiplatform 1.10–1.11":
  - Unified `@Preview` (с 1.10.0) — единая аннотация для всех платформ
  - Shared element improvements (с 1.11.0) — `SharedTransitionScope` улучшения
  - Trackpad events (с 1.11.0) — `detectTrackpadGestures`
  - Coroutine execution in tests (с 1.11.0) — `ComposeUiTest.mainClock` auto-advance
  - WindowInsetsRulers (с 1.10.3)
- Обновлён пример `libs.versions.toml` до актуальных версий (Kotlin 2.4.0, CMP 1.11.0, coroutines 1.11.0, Ktor 3.4.0).
- Структура проекта обновлена: модуль теперь `composeApp` (не `shared`), как в стандарте JetBrains 2026.
- Добавлена секция про Navigation 3 на non-Android targets с примером типобезопасной навигации.
- Добавлено сравнение Navigation Compose / Decompose / Voyager с рекомендациями когда что выбирать.

### Глава 22 — Material Design 3: глубокое погружение (существенное расширение)
**Было:** ~130 строк, M3 Expressive упомянут вскользь, motion physics описан общими словами.  
**Стало:** ~280 строк с подробным разбором M3 Expressive.

**Ключевые изменения:**
- Добавлен раздел "MaterialExpressiveTheme — точка входа" с примером кода.
- Добавлены новые компоненты с полными примерами кода:
  - `LoadingIndicator` и `ContainedLoadingIndicator`
  - `SplitButton` (с `SplitButtonDefaults.SecondaryButton`)
  - `FloatingActionButtonMenu` (с `ExpandableFloatingActionButton` и `FloatingActionButtonMenuItem`)
- Добавлен раздел "22.4. MotionScheme: motion physics" с подробным разбором:
  - Два пресета: `MotionScheme.expressive()` и `MotionScheme.standard()`
  - Пример использования через `MaterialTheme.motionScheme.defaultSpatialSpec()` и `defaultEffectsSpec()`
  - Motion tokens: `md.sys.motion.spring.fast.spatial` и др.
  - Best practice: использовать `MotionScheme.expressive()` по умолчанию
- Добавлено уточнение про переименование `Divider` → `HorizontalDivider` (с 1.4).
- Добавлен пример динамического цвета с проверкой Android 12+ и fallback'ом.

### Глава 29 — Анимации и Motion (точечные правки)
**Было:** MotionScheme не упомянут. В примере с `Animatable` была ошибка — `scope` использовался без объявления.  
**Стало:** Добавлена секция про MotionScheme, исправлена ошибка в коде.

**Ключевые изменения:**
- В описании главы упомянут M3 Expressive (стабилизирован в compose-material3 1.4.x в 2026).
- Полностью переписан раздел 29.6 "M3 Expressive Motion и MotionScheme":
  - Добавлено описание MotionScheme как motion-theming системы
  - Таблица двух пресетов: `expressive` vs `standard`
  - Полный пример кода с `MaterialExpressiveTheme` и `MaterialTheme.motionScheme`
  - Перечислены motion tokens (`md.sys.motion.spring.*`)
- **Исправлена ошибка в примере `Animatable`** (раздел 29.7): добавлен `rememberCoroutineScope()` и `scope.launch { }` обёртки для `animateTo`/`snapTo` внутри `pointerInput` (раньше код не компилировался, т.к. `onDragEnd`/`onHorizontalDrag` не имеют suspend-контекста).
- Добавлен best practice (2026) в конец раздела 29.9: рекомендация использовать `MotionScheme` вместо ручной настройки spring-параметров.

### Глава 32 — Управление состоянием (правки и исправления)
**Было:** В первой строке затесался китайский символ "难以" ("трудноотлаживаемым" было написано как "难以отлаживаемым"). `Snapshot.StateFlow` описан без указания экспериментального статуса.  
**Стало:** Китайский символ удалён, добавлено предупреждение об экспериментальности API.

**Ключевые изменения:**
- **Исправлен китайский символ** в первой строке (заметный баг от GLM 5.1).
- В разделе 32.5 (`Snapshot.StateFlow`) добавлено предупреждение, что API экспериментальный и требует `@OptIn(ExperimentalComposeApi::class)`.
- В разделе 32.4 (`collectAsStateWithLifecycle`) добавлен KMP-нюанс (2026): `collectAsStateWithLifecycle` доступен только на Android, для iOS/Desktop/Wasm Compose Multiplatform автоматически приостанавливает recomposition через `Recomposer.State`.

### Глава 33 — expect/actual: паттерны и антипаттерны (существенное расширение)
**Было:** 5 паттернов + 4 антипаттерна, без упоминания Swift Export.  
**Стало:** Добавлены секции 33.6 (Swift Export и SPM import) и 33.7 (когда НЕ использовать expect/actual).

**Ключевые изменения:**
- В описании главы указана актуальность для Kotlin 2.4.0 с Swift Export (Alpha) и SPM import.
- Добавлена секция "33.6. Swift Export и SPM import (Kotlin 2.4)":
  - Описание Swift Export (Alpha) — генерация Swift-фреймворков минуя Objective-C
  - Пример включения через `gradle.properties`
  - Описание SPM import — объявление Swift packages как зависимостей
  - Пример `spmDependencies { swiftPackage("...") { version = "1.0.0" } }`
  - Раздел "Что это меняет для expect/actual" — имена методов не манглись, suspend → async/await
- Добавлена секция "33.7. Когда НЕ использовать expect/actual":
  - Таблица: задача → старое решение (expect/actual) → современное (готовая библиотека)
  - Правило: предпочитать `multiplatform-settings`, `kotlinx-datetime`, `Napier`, `Kermit` и т.д.

### Глава 34 — KMP Gotchas: подводные камни (правки и обновления)
**Было:** Версии Kotlin 1.7.20/1.8.0, корутины 1.9.0, freeze() описан как "всё ещё нужен в некоторых случаях". Были мелкие опечатки ("kotlinn//", "interop" слитно).  
**Стало:** Версии Kotlin 2.4.0/1.9.0, корутины 1.11.0, freeze() помечен как полностью удалённый.

**Ключевые изменения:**
- Раздел 34.1: версия "стабилен с Kotlin 1.7.20" → "стабилен с Kotlin 1.9, единственный путь с Kotlin 2.0". Добавлено уточнение, что `freeze()` полностью удалён в Kotlin 2.4.
- Раздел 34.2 полностью переписан: заголовок "когда всё ещё нужен?" → "полностью удалён в Kotlin 2.4". Добавлен пример миграции с Kotlin 1.x на 2.4.
- Раздел 34.6: версия корутин 1.9.0 → 1.11.0. Добавлен альтернативный способ через `constraints` в Version Catalog.
- Раздел 34.8: чеклист расширен с 6 до 11 шагов, добавлены KSP2, Compose Multiplatform 1.11.0, kotlinx-coroutines 1.11.0, Ktor 3.4.0, SQLDelight 2.1+, Wasm-билд проверка.
- Добавлен warning в конце: "Не обновляйте Kotlin и Compose Multiplatform в одном PR".

---

## Уровни изменений по категориям

| Категория | Что сделано |
|-----------|-------------|
| **Свежесть версий** | Все версии обновлены до актуальных на 30 июня 2026 (Kotlin 2.4.0, CMP 1.11.0, coroutines 1.11.0, Ktor 3.4.0, Material 3 Expressive). Добавлены конкретные даты релизов. |
| **Точность** | Исправлен китайский символ в главе 32. Исправлена ошибка с `scope` в примере `Animatable` (глава 29). Уточнён статус `Snapshot.StateFlow` (экспериментальный). |
| **Примеры кода** | Добавлены примеры новых M3 Expressive компонентов (LoadingIndicator, SplitButton, FAB Menu). Добавлены примеры MotionScheme. Добавлены примеры Swift Export и SPM import. |
| **Глубина** | Глава 17 расширена с ~50 до ~180 строк. Глава 22 расширена с ~130 до ~280 строк. Глава 33 расширена на 2 новые секции. |

---

## Что НЕ было сделано в этом проходе (для следующей итерации)

Следующие главы пока не обновлялись — это кандидаты на второй проход:

| Глава | Причина приоритета |
|-------|-------------------|
| `01-kotlin-basics.md` | Базовый синтаксис — изменения минимальны, но стоит проверить новые фичи Kotlin 2.4 |
| `02-compose-multiplatform.md` | Установка/Setup — нужно обновить команды и версии |
| `05-data-storage.md` | SQLDelight 2.1+ — проверить новые API |
| `06-networking-api.md` | Ktor 3.4 — проверить изменения SSE, Zstd |
| `08-navigation.md` | Navigation 3 на non-Android — существенное обновление |
| `10-performance.md` | Новый Skia M138, K2 compiler — проверить benchmark'и |
| `11-platform-integration.md` | Swift Export влияет на платформенную интеграцию |
| `13-devops-cicd.md` | KSP2, новые плагины компилятора |
| `14-architecture.md` | MVI/UDF тренды 2026 |
| `20-deep-dive-kotlin.md` | K2, новые языковые фичи Kotlin 2.4 |
| `26-conclusion.md` | Обновить выводы с учётом 2026 |
| `31-adaptive-layouts.md` | WindowSizeClass обновления |
| `37-app-size-optimization.md` | Wasm tree-shaking, R8 изменения |
| metanit/* | Учебник — изменения минимальны, но версии в примерах требуют проверки |
| kotlinlang/* | Перевод официальной документации — требует сверки с kotlinlang.org на актуальность |

---

## Источники, использованные для проверки

- [Kotlin Releases на GitHub](https://github.com/JetBrains/kotlin/releases) — версии 2.4.0, 2.4.10-RC, 2.4.20-Beta1
- [JetBrains Blog — Kotlin 2.4.0 Released](https://blog.jetbrains.com/kotlin/) — Swift packages, Swift export
- [Compose Multiplatform 1.11.0 — JetBrains Blog](https://blog.jetbrains.com/) — UI testing, iOS/web improvements
- [What's new in Compose Multiplatform 1.10.3 — Kotlin](https://kotlinlang.org/) — WindowInsetsRulers, Skia M138
- [Material Design 3 — Motion](https://m3.material.io/) — MotionScheme, motion tokens
- [Adding Motion Physics with Jetpack Compose — Material Design](https://m3.material.io/develop/material-3-expressive) — Expressive vs Standard
- [kotlinx-coroutines на GitHub](https://github.com/Kotlin/kotlinx.coroutines) — 1.11.0 от 9 мая 2026, terminal Flow operators
- [Ktor 3.4.0 — What's new](https://ktor.io/) — Zstd, SSE, Ktor client for Wasm
- [SQLDelight на GitHub](https://github.com/cashapp/sqldelight) — 2.1+
- [KotlinConf 2026 Keynote](https://www.youtube.com/@JetBrainsTV) — Swift Export Alpha, SPM import

---

# Часть 2: актуализация глав с быстро меняющимися API (30 июня 2026)

После первого прохода (главы 17, 21, 22, 29, 32, 33, 34) выполнен второй проход — обновлены главы, где упоминаются часто меняющиеся API библиотек и инструментов. Метанит и kotlinlang-переводы не трогались (по решению пользователя — только для контекста).

## Сводка изменений второго прохода

### Глава 02 — Compose Multiplatform (setup/установка)
**Было:** IntelliJ IDEA 2024.2+, плагин 1.7.3, Kotlin 2.1.0.  
**Стало:** IntelliJ IDEA 2026.1+, ссылка на KMP Web Wizard, Kotlin 2.4.0, CMP 1.11.0, обновлённая структура `composeApp/`.

**Ключевые изменения:**
- Добавлена ссылка на [KMP Web Wizard](https://kmp.jetbrains.com/) — рекомендованный путь создания проекта.
- Обновлены требования к IDE: IntelliJ IDEA 2026.1+, JDK 17+ (рекомендуется 21 LTS).
- Версии плагинов: Kotlin 2.4.0, `org.jetbrains.kotlin.plugin.compose` (выделенный артефакт с Kotlin 2.0+), CMP 1.11.0.
- Добавлено пояснение про стандартизацию `composeApp` (а не `shared`) с CMP 1.10.
- Добавлено предупреждение о таблице совместимости версий Kotlin и CMP.

### Глава 05 — Работа с данными (полная переработка)
**Было:** Устаревший groupId `com.squareup.sqldelight` (1.x), упрощённый пример без транзакций/миграций/Flow, краткое упоминание multiplatform-settings.  
**Стало:** Полноценный гайд по SQLDelight 2.1+ с правильным groupId `app.cash.sqldelight`, миграциями, Flow-наблюдениями, multiplatform-settings.

**Ключевые изменения:**
- **Исправлен критический баг:** groupId SQLDelight изменён с `com.squareup.sqldelight` на `app.cash.sqldelight` (с версии 2.0).
- Добавлены все платформенные драйверы: `AndroidSqliteDriver`, `NativeSqliteDriver`, `JdbcSqliteDriver`.
- Добавлен пример `sqldelight-coroutines-extensions` — Flow-наблюдение за изменениями (`queries.selectAll().asFlow().asList()`).
- Добавлен раздел про миграции через `.sqm` файлы.
- Добавлен раздел про multiplatform-settings 1.2.0 с платформенными реализациями (SharedPreferencesSettings, NSUserDefaultsSettings, localStorageSettings, DataStoreSettings).
- Добавлен полноценный раздел про Offline-first паттерн с разрешением конфликтов (LWW, Versioned, Merge, Manual).

### Глава 06 — Интеграция с API (полная переработка)
**Было:** Устаревший API `install(JsonFeature)` (deprecated с Ktor 2.0, удалён в 3.0). `ktor-client-cio` для iOS (на самом деле для iOS используется Darwin engine). Поверхностные примеры без обработки ошибок.  
**Стало:** Полноценный гайд по Ktor 3.4 с правильным `ContentNegotiation`, Bearer Auth с `refreshTokens`, обработкой ошибок.

**Ключевые изменения:**
- **Исправлен критический баг:** `install(JsonFeature) { serializer = KotlinxSerializer() }` → `install(ContentNegotiation) { json(Json { ... }) }` (JsonFeature удалён в Ktor 3.0).
- **Исправлен баг с engine'ами:** для iOS указан `ktor-client-darwin` (а не `ktor-client-cio`).
- Добавлены все платформенные engine'ы: `OkHttp` (Android), `Darwin` (iOS), `Java` (Desktop), `Js` (Wasm/JS).
- Добавлена конфигурация `Json { ignoreUnknownKeys = true; explicitNulls = false }` — критично для forward-compatibility.
- Добавлен раздел 6.3 «Обработка ошибок» с иерархией исключений Ktor: `ClientRequestException`, `ServerResponseException`, `HttpRequestTimeoutException`, `RedirectResponseException`, `JsonConvertException`.
- Добавлен пример retry с экспоненциальной задержкой и упоминание встроенного плагина `HttpRetry` (Ktor 3.1+).
- Существенно расширен раздел аутентификации: Bearer Auth с `loadTokens`/`refreshTokens`/`sendWithoutRequest`, Basic Auth, API ключи.
- Добавлен раздел 6.5 «Загрузка файлов (multipart)» с примером `submitFormWithBinaryData`.
- Добавлен раздел 6.6 «WebSocket (real-time данные)» с примером `webSocket { sendSerialized/receiveDeserialized }`.
- Добавлен раздел 6.7 «Best practices» — таблица типичных сценариев и решений.

### Глава 08 — Навигация (полная переработка)
**Было:** Устаревший API `route = "details/{taskId}"` (строки вместо type-safe). Упоминание Decompose 3.2.0, без Navigation 3.  
**Стало:** Type-safe навигация с Navigation Compose 2.9+, обновлённый Decompose 3.3, добавлено сравнение библиотек.

**Ключевые изменения:**
- **Полностью переписан раздел 8.1:** вместо строковых маршрутов — `@Serializable` классы и объекты (`TaskDetailRoute(val taskId: String)`), `composable<TaskDetailRoute>`, `backStackEntry.toRoute<TaskDetailRoute>()`.
- Добавлен пример передачи сложных параметров через `@Serializable` структуры.
- Добавлен пример анимаций переходов (`enterTransition`, `exitTransition`, `popEnterTransition`, `popExitTransition`).
- Добавлен пример Bottom navigation с вложенными стеками.
- Обновлён Decompose с 3.2.0 до 3.3.0, переписан пример с правильным API (`childStack` вместо `ChildStack`).
- Добавлено сравнение Navigation Compose / Decompose / Voyager с рекомендациями когда что выбирать.
- Добавлен раздел 8.5 «Глубокие ссылки» с примером `navDeepLink<TodoRoute>(basePath = "myapp://tasks")` для type-safe навигации.
- Добавлен раздел 8.6 «Сводная таблица выбора навигационной библиотеки».

### Глава 10 — Производительность (расширение)
**Было:** Базовая информация про recompositions, без упоминания contentType, без Skia M144.  
**Стало:** Добавлены `contentType` в Lazy-списках, секция про Skia M144, K2 compiler.

**Ключевые изменения:**
- Добавлен `contentType` в примере `items(tasks, key = { it.id }, contentType = { it.type })` — важно для гетерогенных списков, снижает количество создаваемых компонентов.
- `Modifier.animateItemPlacement()` → `Modifier.animateItem()` (с Compose 1.7+).
- Добавлен раздел 10.6 «Рендеринг через Skia M144» с информацией о версии Skia в CMP 1.11.1, backend'ах рендеринга, проверке версии skiko.
- Добавлен раздел 10.7 «K2 compiler и производительность сборки» с упоминанием KSP2 (до 2x быстрее annotation processing).

### Глава 20 — Глубокое погружение в Kotlin (расширение)
**Было:** Базовое описание K2, без новых языковых фич Kotlin 2.x.  
**Стало:** Добавлены Kotlin 2.2/2.4 языковые фичи, KSP2, Swift Export, Wasm GC.

**Ключевые изменения:**
- Добавлена секция «K2 и плагины: KSP2» с примером `id("com.google.devtools.ksp") version "2.4.0-1.0.21"`.
- Добавлена секция «Миграция на K2» с описанием breaking changes и ссылкой на K2 Migration Guide.
- Исправлен устаревший пример: `String.capitalize()` (deprecated с 1.5) → `replaceFirstChar { it.uppercase() }`.
- Добавлен раздел 20.5 «Новые возможности Kotlin 2.x»:
  - **Guard-выражения** (Kotlin 2.2 stable) — `when` с `if`-условиями.
  - **Non-local break и continue** (Kotlin 2.2 stable) — в inline-лямбдах.
  - **Multi-dollar string literals** (Kotlin 2.2 stable) — `$$price` для экранирования `$`.
  - **Контекстные параметры** (Kotlin 2.4 — Alpha) — эволюция контекстных ресиверов.
- Добавлен раздел 20.6 «Kotlin 2.4 — что нового для Multiplatform»: Swift Export, SPM import, стабилизация Wasm GC, coroutine stack trace recovery (2.4.20-Beta1).

### Глава 26 — Заключение и перспективы (полная переработка)
**Было:** Общие выводы про «зрелый стек» без конкретики 2026, упоминание «iOS stable (май 2025) и Web beta (сентябрь 2025)» — устаревшая информация.  
**Стало:** Актуальные выводы на 30 июня 2026, с конкретными фичами и датами, рекомендации по обучению.

**Ключевые изменения:**
- Обновлены все статусы: iOS — Stable (зрелость в 1.11.0), Web (Wasm) — Stable-ready (с 1.11.0).
- Добавлены конкретные новые возможности 2026 года для каждой технологии.
- Добавлен раздел «Стратегии миграции» с тремя уровнями (только бизнес-логика, UI для части экранов, полная кроссплатформенность).
- Добавлены перспективы развития (краткосрочные, среднесрочные, долгосрочные тренды).
- Добавлены рекомендации «Когда выбирать стек» и «Когда не рекомендуется».
- Добавлен раздел «Стратегия обучения для тех, кто переходит на Kotlin» — 5-шаговый путь с конкретными ссылками.

### Глава 09 — Тестирование (правки)
**Было:** В целом актуально (Mockative 2.4.1, Kover 0.9.1). Мелкие проблемы: неконсистентное использование `runTest`.  
**Стало:** Без существенных изменений — содержимое корректное, актуальные версии библиотек.

**Проверено:** Mockative 2.4.1 — актуальная версия; Kover 0.9.1 — актуальная; kotlin-test — стандарт. Глава не требовала обновления.

### README.md (обновление)
- Заголовок: добавлены версии Kotlin 2.4, CMP 1.11, Material 3 Expressive, дата актуализации.
- Обновлён Quick Start: добавлена ссылка на KMP Web Wizard.
- Обновлён стек технологий: конкретные версии + groupId SQLDelight `app.cash.sqldelight` + Navigation 2.9+ type-safe.
- Добавлена ссылка на главу 17 (Обновления экосистемы) для полной таблицы версий.

---

## Сводная таблица всех изменений за оба прохода

| Глава | Тип изменения | Ключевое |
|-------|---------------|----------|
| 02 | Точечные правки | KMP Wizard, Kotlin 2.4.0, CMP 1.11.0, `org.jetbrains.kotlin.plugin.compose` |
| 05 | Полная переработка | SQLDelight `app.cash.sqldelight` 2.1+, multiplatform-settings, Offline-first |
| 06 | Полная переработка | `ContentNegotiation` (вместо JsonFeature), Bearer Auth, обработка ошибок |
| 08 | Полная переработка | Type-safe Navigation Compose 2.9+, Decompose 3.3 |
| 10 | Расширение | `contentType`, Skia M144, K2 compiler |
| 17 | Полная переработка | Таблица версий, M3 Expressive, тренды 2026 |
| 20 | Расширение | KSP2, Kotlin 2.2/2.4 языковые фичи, Swift Export |
| 21 | Полная переработка | CMP 1.11, Unified @Preview, Navigation 3, новые фичи |
| 22 | Существенное расширение | M3 Expressive, MotionScheme, новые компоненты |
| 26 | Полная переработка | Актуальные выводы 2026, рекомендации по обучению |
| 29 | Точечные правки | MotionScheme, исправление ошибки с scope в Animatable |
| 32 | Правки | Удалён китайский символ, уточнён Snapshot.StateFlow статус |
| 33 | Существенное расширение | Swift Export, SPM import, таблица «когда НЕ использовать expect/actual» |
| 34 | Правки | Kotlin 2.4, freeze() удалён, чеклист 11 шагов |
| README | Обновление | Актуальные версии, KMP Wizard, стек технологий |

**Всего обновлено:** 14 глав + README + CHANGELOG.

---

## Источники, использованные для проверки (часть 2)

- [Ktor — Migrating from 1.6.x to 2.0.x](https://ktor.io/docs/migration.html) — JsonFeature → ContentNegotiation
- [Ktor 3.4.0 — What's new](https://ktor.io/docs/whatsnew_34.html) — Zstd, SSE, fallback в OAuth
- [SQLDelight на GitHub](https://github.com/cashapp/sqldelight) — groupId `app.cash.sqldelight` с 2.0
- [Type safety in Navigation Compose](https://developer.android.com/guide/navigation/type-safety) — type-safe routes с 2.8.0
- [Compose Multiplatform 1.11.1 — What's new](https://kotlinlang.org/docs/whatsnew-compose-multiplatform-1111.html) — Skia M144
- [Kotlin 2.4.0 Released — JetBrains Blog](https://blog.jetbrains.com/kotlin/2026/05/15/kotlin-2-4-0-released/) — Swift Export, SPM import
- [What's new in Kotlin 2.4.0](https://kotlinlang.org/docs/whatsnew20.html) — контекстные параметры, coroutine stack trace recovery
- [Kotlin 2.2 Livestream — Language Evolution Team](https://www.youtube.com/watch?v=) — guards, non-local break/continue, multi-dollar strings
- [Mocking in KMP — Mockative 2](https://github.com/mockative/mockative) — Mockative 2.x для KMP
- [multiplatform-settings на GitHub](https://github.com/russhwolf/multiplatform-settings) — версия 1.2.0
