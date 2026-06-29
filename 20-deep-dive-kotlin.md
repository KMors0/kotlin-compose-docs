# Глубокое погружение в Kotlin

> Расширенное рассмотрение системы типов, корутин, компилятора K2 и архитектуры Kotlin Multiplatform — с опорой на реальные данные и исследования.

---

## 20.1. Система типов и Null Safety

Одной из фундаментальных особенностей Kotlin является встроенная null-safety на уровне системы типов. В отличие от Java, где любая ссылочная переменная может быть `null` (что приводит к `NullPointerException` — самому частому исключению в Java-приложениях), Kotlin разделяет типы на nullable (`String?`) и non-nullable (`String`).

```kotlin
val name: String = "Kotlin"          // Не может быть null
val nickname: String? = null          // Может быть null

// Безопасный вызов
val length: Int? = nickname?.length   // Возвращает null, если nickname == null

// Оператор Элвиса
val displayName: String = nickname ?: "Anonymous"

// Приведение с проверкой (smart cast)
if (nickname != null) {
    println(nickname.length)          // Компилятор автоматически приводит к String
}
```

Компилятор Kotlin отслеживает поток данных (data flow analysis) и предотвращает потенциально опасные операции с null на этапе компиляции. Это практически исключает `NullPointerException` в корректном Kotlin-коде, что по данным JetBrains экономит разработчикам до 20% времени на отладку.

Дополнительные возможности системы типов включают:
- **Smart casts** — автоматическое приведение типов после проверки
- **Type aliases** — псевдонимы типов для сложных обобщений
- **Reified type parameters** — сохранение информации о типе в runtime для inline-функций
- **Sealed classes и interfaces** — ограниченные иерархии наследования для исчерпывающего `when`-выражения
- **Inline value classes** — обёртки над примитивными типами с нулевой стоимостью в runtime

---

## 20.2. Корутины и структурированная конкуренция

Корутины (coroutines) — встроенный в стандартную библиотеку Kotlin механизм для написания асинхронного кода в последовательном стиле. В отличие от потоков операционной системы, корутины являются легковесными — можно создать сотни тысяч корутин без значительных затрат памяти, поскольку они не привязаны к конкретному потоку и могут приостанавливать (suspend) своё выполнение, освобождая поток для других корутин.

```kotlin
// Базовый пример корутины
suspend fun fetchUserData(): User {
    val profile = apiService.getProfile()    // Приостанавливается, ожидая сеть
    val settings = apiService.getSettings()  // Не блокирует поток
    return User(profile, settings)
}

// Запуск в параллель с structured concurrency
suspend fun loadDashboard() = coroutineScope {
    val deferredProfile = async { apiService.getProfile() }
    val deferredFeed = async { apiService.getFeed() }
    Dashboard(deferredProfile.await(), deferredFeed.await())
}
```

**Структурированная конкуренция (Structured Concurrency)** — паттерн, при котором корутины организуются в иерархию «родитель-потомок». Если родительская корутина отменяется или завершается с ошибкой, все её дочерние корутины автоматически отменяются. Это обеспечивает предсказуемое управление жизненным циклом асинхронных операций и предотвращает утечки ресурсов.

По данным Android Developer Documentation, более 50% профессиональных разработчиков, использующих корутины, отмечают повышение продуктивности. Корутины стали стандартом для асинхронного программирования на Android, заменив громоздкие `AsyncTask`, `RxJava` и callback-based подходы.

### Основные строительные блоки корутин

| Блок | Описание |
|------|----------|
| **Dispatchers** | Определяют, на каком пуле потоков выполняется корутина (`Main`, `IO`, `Default`, `Unconfined`) |
| **Job** | Дескриптор выполняемой корутины, поддерживающий жизненный цикл, отмену и родительские/дочерние отношения |
| **Flow** | Холодный асинхронный поток данных, аналог Reactor's Flux или RxJava's Observable, но интегрированный с корутинами |

---

## 20.3. Компилятор K2

Kotlin 2.0 (май 2024) представил новый компилятор K2 с полностью переписанным фронтендом, использующим единую структуру данных с расширенной семантической информацией. K2 обеспечивает:

- **Ускорение компиляции:** до 94% ускорение чистых сборок по сравнению с K1 (по результатам бенчмарков Tech Insider)
- **Улучшенное разрешение вызовов и вывод типов:** компилятор более последовательно понимает код разработчика
- **Упрощённое введение синтаксического сахара:** новые возможности языка будут добавляться быстрее
- **Улучшенная производительность IDE:** более быстрая подсветка синтаксиса, автодополнение и анализ кода
- **Унификация платформ:** единый компилятор для всех целевых платформ (JVM, JS, Native, Wasm)

### K2 и плагины: KSP2

Начиная с Kotlin 2.4, рекомендуется использовать **KSP2** — новое поколение Kotlin Symbol Processing API. KSP2 работает на базе K2, существенно быстрее (до 2x в некоторых проектах) и поддерживает incremental processing лучше, чем KSP1.

```kotlin
// build.gradle.kts
plugins {
    id("com.google.devtools.ksp") version "2.4.0-1.0.21"  // KSP2
}
```

> **Миграция с KSP1:** в большинстве случаев достаточно обновить версию KSP до совместимой с Kotlin 2.4. KSP2 обратно совместим с большинством KSP1-процессоров. Известные проблемы — с процессорами, использующими internal API.

### Миграция на K2

Миграция на K2 прошла относительно гладко для большинства проектов, хотя некоторые проблемы были связаны с изменениями в поведении вывода типов и отражения (reflection). Главные Breaking changes:

- **Более строгий вывод типов в лямбдах** — некоторые паттерны, которые работали в K1 из-за слабого вывода, теперь требуют явных аннотаций.
- **Изменения в `@JvmDefault`** — теперь всегда включено, аннотация deprecated.
- **Унифицированная обработка `when`-выражений** — exhaustive check теперь строже.

JetBrains предоставляет [K2 Migration Guide](https://kotlinlang.org/docs/k2-migration-guide.html) с подробным списком изменений.

---

## 20.4. Архитектура Kotlin Multiplatform (KMP)

Kotlin Multiplatform позволяет делить код на **общий** (shared) и **платформоспецифичный** (platform-specific). Общий код компилируется для всех целевых платформ, а платформоспецифичный — только для конкретной платформы через механизм `expect`/`actual`:

```kotlin
// Общий модуль (commonMain)
expect fun platformName(): String

// Модуль Android (androidMain)
actual fun platformName(): String = "Android"

// Модуль iOS (iosMain)
actual fun platformName(): String = "iOS"
```

Новая структура проекта по умолчанию для KMP, объявленная JetBrains в мае 2026 года, даёт модулям более чёткие ответственности, лучше согласуется с конвенциями других систем сборки и отражает изменения в Android Gradle Plugin 9.0. Типичная структура включает: `shared/` — общий модуль с бизнес-логикой и UI, `androidApp/` — точка входа для Android, `iosApp/` — точка входа для iOS (Xcode проект), `desktopApp/` и `webApp/` — для десктопа и веба соответственно.

---

## 20.5. Ключевые возможности языка

### Функции расширения (Extension Functions)

Позволяют добавлять функциональность к классам без наследования или паттерна Decorator:

```kotlin
fun String.capitalizeWords(): String =
    split(" ").joinToString(" ") { it.replaceFirstChar { c -> c.uppercase() } }

// Использование
"hello world".capitalizeWords()  // "Hello World"
```

> **Примечание:** `String.capitalize()` устарел с Kotlin 1.5, используйте `replaceFirstChar { it.uppercase() }`.

### Data classes

Автоматически генерируют `equals()`, `hashCode()`, `toString()`, `copy()` и `componentN()` функции, уменьшая бойлерплейт:

```kotlin
data class User(val name: String, val age: Int)
val user = User("Alice", 30)
val updated = user.copy(age = user.age + 1)
```

### Новые возможности Kotlin 2.x

#### Guard-выражения (Kotlin 2.2 stable, было в preview с 2.1)

`when`-выражения поддерживают guard-условия через `if`:

```kotlin
fun classify(number: Int): String = when (number) {
    0 -> "ноль"
    else if (number > 0) -> "положительное"
    else if (number < 0) -> "отрицательное"
    else -> "невозможно"  // на самом деле недостижимо
}

// Полезно для sealed-классов с дополнительными условиями
when (state) {
    is State.Loading -> "Загрузка"
    is State.Success if state.data.isEmpty() -> "Пусто"
    is State.Success -> "Данные: ${state.data.size}"
    is State.Error if state.isRetryable -> "Ошибка, можно повторить"
    is State.Error -> "Ошибка"
}
```

#### Non-local break и continue (Kotlin 2.2 stable)

В лямбдах, переданных в inline-функции, можно использовать `break` и `continue` для выхода из внешнего цикла:

```kotlin
for (i in 1..10) {
    listOf(1, 2, 3, -1, 4).forEach {
        if (it < 0) break  // выходит из for, а не только из forEach
        println("$i - $it")
    }
}
```

#### Multi-dollar string literals (Kotlin 2.2 stable)

Для строк, содержащих символ `$`, можно использовать несколько `$$` для экранирования:

```kotlin
val template = """
    Price: $$price
    Variable: \\$variable
"""".trimIndent()
// $$price → буквально $price, \\$variable → ${'$'}variable
```

#### Контекстные параметры (Kotlin 2.4 — Alpha)

Контекстные параметры — эволюция контекстных ресиверов, позволяют объявить, что функция требует определённый контекст вызова:

```kotlin
context(logger: Logger)
fun doSomething() {
    logger.info("Doing something")
}

// Использование
with(Logger.STDOUT) {
    doSomething()  // компилятор подставит logger автоматически
}

// Явная передача (Kotlin 2.4+)
doSomething(logger = Logger.STDOUT)
```

> **Статус (Kotlin 2.4.0):** контекстные параметры в Alpha-статусе. Поддержка в IntelliJ IDEA 2026.2. Для production пока лучше использовать классические подходы (DI через Koin, параметры функции).

### Другие возможности

- **Классы-делегаты (Delegation)** через ключевое слово `by` — реализуют паттерн «делегирование» без бойлерплейт-кода
- **DSL-конструкторы** — позволяют создавать предметно-ориентированные языки, как это делает сам Compose с декларативным UI
- **Sealed классы и интерфейсы** — обеспечивают исчерпывающую обработку вариантов в `when`-выражениях
- **Inline-функции с reified-параметрами** — сохраняют информацию о типах в runtime
- **Деструктурирующие объявления** — позволяют распаковывать объекты на отдельные переменные
- **Value classes (inline)** — обёртки над примитивными типами с нулевой стоимостью в runtime

---

## 20.6. Kotlin 2.4 — что нового для Multiplatform

Кotlin 2.4 (май 2026) добавил несколько важных возможностей для KMP-разработчиков:

### Swift Export (Alpha)

Новая генерация iOS-фреймворков напрямую в Swift, минуя Objective-C. См. раздел [33. expect/actual паттерны](33-expect-actual-patterns.md#336-swift-export-и-spm-import-kotlin-24).

### Swift Package Manager import

Возможность объявлять Swift packages как зависимости KMP-модуля. Раньше для использования Swift-библиотеки из Kotlin/Native нужно было создавать Objective-C wrapper, теперь можно напрямую:

```kotlin
kotlin {
    iosArm64()
    spmDependencies {
        swiftPackage("https://github.com/example/SwiftLib.git") {
            version = "1.0.0"
        }
    }
}
```

### Стабилизация Wasm GC

Wasm-таргет (Kotlin/Wasm) теперь полностью стабильный. Compose Multiplatform 1.11+ использует Wasm GC для рендеринга UI в браузере, что обеспечивает производительность на уровне нативных приложений.

### Coroutine stack trace recovery (Kotlin 2.4.20-Beta1)

В standard library добавлена поддержка восстановления stack trace в корутинах — ранее при ошибке в корутине stack trace не показывал полную цепочку вызовов. Это улучшение упрощает отладку асинхронного кода.

---

⬅️ [← Введение](00-introduction.md) | [К содержанию](README.md) | [Глубокое погружение в Compose →](21-deep-dive-compose.md) ➡️
