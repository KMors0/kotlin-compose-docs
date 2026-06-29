# KMP Gotchas: подводные камни

> Переход на Kotlin Multiplatform — это не просто «написать код на Kotlin». Есть ряд неочевидных проблем, на которые спотыкаются почти все команды. Этот раздел описывает реальные gotchas из production-проектов и способы их решения.

---

## 34.1. Новый Memory Manager ( стабилен с Kotlin 1.9, полностью единственный путь с Kotlin 2.0)

**Проблема:** Старый memory manager в Kotlin/Native использовал reference counting с циклическим сборщиком (GC). Это приводило к:
- Задержкам сборки мусора (GC pauses до 100 мс) на главном потоке.
- Утечкам памяти при циклических ссылках между Kotlin и Swift/Objective-C объектами.
- Необходимости вручную управлять временем жизни объектов через `freeze()`.

**Решение:** Новый memory manager (New MM) использует автоматический сборщик мусора, аналогичный JVM. Он стабилен с Kotlin 1.9.0 и **единственный поддерживаемый вариант с Kotlin 2.0** (старый MM полностью удалён).

**Что значит для вас:**
```kotlin
// С Новым MM: НЕ НУЖНО замораживать объекты
val task = Task("1", "Buy milk", false)
// Без freeze() — объект мутабелен, но безопасность потоков гарантирует GC

// С Новым MM: НЕ НУЖНО использовать WeakReference для разрыва циклов
// GC сам обнаружит и соберёт циклические ссылки
```

**Но есть нюансы:**
- На iOS межпоточные операции всё ещё требуют осторожности — GC работает на каждом потоке отдельно.
- Shared mutable state без синхронизации — UB даже с Новым MM (используйте `AtomicReference`, `@SharedImmutable`, или корутины с `Mutex`).
- `freeze()` полностью удалён в Kotlin 2.4 — вызов `kotlin.native.concurrent.freeze()` теперь deprecated и выбрасывает исключение в Kotlin 2.x.

---

## 34.2. freeze() — полностью удалён в Kotlin 2.4

В Kotlin 1.9 `freeze()` был deprecated, в Kotlin 2.0 — выбрасывал исключение, в Kotlin 2.4 — полностью удалён из API. Если вы обновляетесь с Kotlin 1.x, вам необходимо удалить все вызовы `freeze()`.

**Сценарии, где раньше требовался freeze():**

**1. Передача объектов в фоновые потоки (Legacy API):**
```kotlin
// Kotlin 1.x (устарело):
val frozenList = listOf(1, 2, 3).freeze()
dispatch_async(backgroundQueue) { println(frozenList) }

// Kotlin 2.4 (современно):
val list = listOf(1, 2, 3)  // Ничего не замораживаем
val scope = CoroutineScope(Dispatchers.Default)
scope.launch { println(list) }  // Работает из коробки
```

**2. Interop с Swift structs:** Swift value types копируются — при передаче в Kotlin/Native они работают как обычно, заморозка не требуется.

**3. Глобальные константы:**
```kotlin
// Global mutable state — осторожно
var globalCache = mutableMapOf<String, String>()
// Без синхронизации — UB при доступе из разных потоков
// Решение: kotlinx.atomicfu или @SharedImmutable для immutable данных

@SharedImmutable  // Делает immutable копию при инициализации
val sharedConfig: Map<String, String> = mapOf("key" to "value")
```

---

## 34.3. AtomicFU — атомарные операции в KMP

[AtomicFU](https://github.com/Kotlin/atomicfu) — библиотека атомарных операций, которую использует сам kotlinx.coroutines внутри. Для прикладного кода она нужна, когда:
- Вы реализуете свой State-менеджер.
- Нужен thread-safe счётчик или флаг.
- Вы пишете библиотеку.

```kotlin
import kotlinx.atomicfu.atomic
import kotlinx.atomicfu.updateAndGet

class AtomicTaskCounter {
    private val _count = atomic(0)
    val count: Int get() = _count.value

    fun increment() { _count.incrementAndGet() }
    fun decrement() { _count.decrementAndGet() }
}
```

> **Для UI-состояния в Compose используйте `mutableStateOf()` — он уже атомарен через Snapshot.** AtomicFU нужен только для state вне Compose.

---

## 34.4. Serialization и polymorphism

**Проблема:** `kotlinx.serialization` не поддерживает полиморфную сериализацию в общих модулях KMP без дополнительных шагов.

```kotlin
// commonMain
@Serializable
sealed interface NetworkResult<T> {
    @Serializable
    data class Success<T>(val data: T) : NetworkResult<T>
    @Serializable
    data class Error(val message: String, val code: Int) : NetworkResult<T>
}
```

**Gotcha:** Если `NetworkResult` используется в expect/actual контексте, генератор сериализации может не найти все подклассы. Решение — использовать `@SerializersModule` и явную регистрацию:

```kotlin
val module = SerializersModule {
    polymorphic(NetworkResult::class) {
        subclass(NetworkResult.Success::class)
        subclass(NetworkResult.Error::class)
    }
}

val json = Json { serializersModule = module }
```

---

## 34.5. iOS interop: утечки retain cycle

**Проблема:** Kotlin/Native объекты и Swift объекты ссылаются друг на друга → retain cycle → утечка памяти.

```kotlin
// ❌ Утечка: Swift-объект держит Kotlin-объект, Kotlin-объект держит Swift-объект
class KotlinDelegate {
    var swiftCallback: ((String) -> Unit)? = null  // Держит Swift-лямбду
}

// Swift side:
let delegate = KotlinDelegate()
let viewModel = MyViewModel(delegate: delegate)
// delegate.viewModel = viewModel // CYCLE!
```

**Решение:** Используйте `weak` ссылки (на Swift-стороне) или `WeakReference` (на Kotlin-стороне):

```swift
// Swift — используйте [weak self]
viewModel.onResult = { [weak self] result in
    self?.handleResult(result)
}
```

---

## 34.6. Gradle: транзитивные зависимости и конфликты

**Gotcha:** Разные KMP-библиотеки могут зависеть от разных версий `kotlinx-coroutines`. Это приводит к runtime-крашам на iOS.

```kotlin
// build.gradle.kts — принудительная унификация
configurations.all {
    resolutionStrategy.eachDependency {
        if (requested.group == "org.jetbrains.kotlinx" &&
            requested.name.startsWith("kotlinx-coroutines")) {
            useVersion("1.11.0")  // актуальная версия на 2026-06-30
        }
    }
}
```

**Альтернативный способ через `constraints` в Version Catalog:**
```kotlin
dependencies {
    constraints {
        implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.11.0")
        implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.8.0")
    }
}
```

**Инструмент:** `./gradlew :composeApp:dependencies --configuration iosArm64CompileKlibs` — проверьте дерево зависимостей для каждой платформы.

---

## 34.7. Строки и кодировки

**Gotcha:** На iOS строковые константы из Objective-C фреймворков могут содержать null-байты. `String.toByteArray()` в Kotlin/Native ведёт себя иначе, чем на JVM.

```kotlin
// commonMain — безопасная конвертация
fun String.toUtf8Bytes(): ByteArray = encodeToByteArray()

fun ByteArray.fromUtf8String(): String = decodeToString()
// Эти функции kotlinx сериализации и работают корректно на всех платформах.
```

---

## 34.8. Чеклист миграции на Kotlin 2.4 + Новый MM

Если вы мигрируете проект с Kotlin 1.x на Kotlin 2.4:

| Шаг | Действие | Проверка |
|------|---------|----------|
| 1 | Обновить Kotlin до 2.4.0 в `libs.versions.toml` | `./gradlew :composeApp:dependencies` |
| 2 | Обновить KSP до 2.x (KSP2, рекомендуется) | Проверить совместимость в plugin |
| 3 | Удалить все вызовы `freeze()` | Глобальный поиск по проекту |
| 4 | Заменить `Worker` на `CoroutineDispatcher` | Нет вызовов `execute` на фоне |
| 5 | Заменить `AtomicReference` на `atomicfu` | Если использовали для state |
| 6 | Обновить Compose Multiplatform до 1.11.0 | Проверить `compose.kotlin.native.mode=native` |
| 7 | Обновить kotlinx-coroutines до 1.11.0 | Версия из `libs.versions.toml` |
| 8 | Обновить Ktor до 3.4.0, SQLDelight до 2.1+ | Версии из `libs.versions.toml` |
| 9 | Запустить все тесты | `./gradlew :composeApp:allTests` |
| 10 | Профилировать память на iOS | Instruments → Allocations |
| 11 | Проверить Wasm-билд (если есть) | `./gradlew :composeApp:wasmJsBrowserProductionRun` |

> **Важно:** Не обновляйте Kotlin и Compose Multiplatform в одном PR. Сначала Kotlin → проверка → потом CMP → проверка. Если что-то сломается, будет сложно понять, кто виноват.

---

⬅️ [← expect/actual паттерны](33-expect-actual-patterns.md) | [К содержанию](README.md) | [Адаптивные лейауты →](31-adaptive-layouts.md) ➡️
