# expect/actual: паттерны и антипаттерны

> Механизм `expect`/`actual` — фундаментальная идиома Kotlin Multiplatform. Он позволяет объявлять API в общем модуле и предоставлять платформенные реализации. Этот раздел описывает реальные паттерны из production-проектов и частые ошибки, которые приводят к поддерживаемому или, наоборот, к хрупкому коду. Все примеры актуальны для Kotlin 2.4.0 с поддержкой Swift Export (Alpha) и SPM import.

---

## 33.1. Что можно объявлять как expect

| Конструкция | Поддерживается | Примечание |
|-------------|---------------|------------|
| Функции (top-level, extension) | ✅ | Самый частый случай |
| Свойства (val/var) | ✅ | Включая делегированные |
| Классы | ✅ | С ограничениями: нельзя менять иерархию наследования |
| Интерфейсы | ✅ | actual может расширять дополнительные интерфейсы |
| Объекты (object) | ✅ | Singleton-паттерн |
| Аннотации | ✅ | С ограничениями на параметры |
| Enum-классы | ✅ (с 1.9) | В ранних версиях — только через workaround |
| Type aliases | ✅ | Полезно для платформенных типов |
| Sealed-классы | ❌ | Невозможно — используйте обычный sealed в commonMain |

---

## 33.2. Паттерны

### Паттерн 1: expect class с конструктором

Самый частый паттерн — класс, инкапсулирующий платформенный API.

```kotlin
// commonMain
expect class PlatformLogger() {
    fun d(tag: String, message: String)
    fun e(tag: String, message: String, throwable: Throwable? = null)
    fun w(tag: String, message: String)
}

// androidMain
actual class PlatformLogger {
    actual fun d(tag: String, message: String) {
        if (BuildConfig.DEBUG) Log.d(tag, message)
    }
    actual fun e(tag: String, message: String, throwable: Throwable?) {
        Log.e(tag, message, throwable)
    }
    actual fun w(tag: String, message: String) {
        Log.w(tag, message)
    }
}

// iosMain
actual class PlatformLogger {
    actual fun d(tag: String, message: String) {
        NSLog("[$tag] $message")
    }
    actual fun e(tag: String, message: String, throwable: Throwable?) {
        NSLog("[$tag] ERROR: $message ${throwable?.localizedMessage ?: ""}")
    }
    actual fun w(tag: String, message: String) {
        NSLog("[$tag] WARN: $message")
    }
}
```

### Паттерн 2: expect fun без класса (функциональный стиль)

Для простых случаев не нужно создавать класс — достаточно функции.

```kotlin
// commonMain
expect fun getCurrentTimestampMs(): Long
expect fun formatFileSize(bytes: Long): String

// androidMain
actual fun getCurrentTimestampMs() = System.currentTimeMillis()
actual fun formatFileSize(bytes: Long): String {
    return Formatter.formatShortFileSize(context, bytes)
}

// iosMain
actual fun getCurrentTimestampMs() = NSDate().timeIntervalSince1970.toLong() * 1000
actual fun formatFileSize(bytes: Long): String {
    val formatter = NSByteCountFormatter()
    return formatter.stringFromByteCount(bytes.toLong())
}
```

### Паттерн 3: expect class как фабрика (DI-friendly)

Вместо создания объекта в `expect` классе, внедряйте зависимость через конструктор. Это делает код тестируемым.

```kotlin
// commonMain
expect class SecureStorage(context: PlatformContext) {
    fun save(key: String, value: String)
    fun read(key: String): String?
    fun delete(key: String)
    fun contains(key: String): Boolean
}

// androidMain
actual class SecureStorage(context: PlatformContext) {
    private val prefs = EncryptedSharedPreferences.create(
        "secure_prefs", MasterKey.Builder(context).build(),
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM,
        context
    )
    // ...implementations
}

// iosMain
actual class SecureStorage(context: PlatformContext) {
    private val keychain = KeychainAccessor()
    // ...implementations using Security.framework
}
```

### Паттерн 4: expect object для синглтонов

Системные сервисы, доступные в одном экземпляре — кандидат на `expect object`.

```kotlin
// commonMain
expect object FileSystem {
    fun readFile(path: String): ByteArray
    fun writeFile(path: String, data: ByteArray)
    fun deleteFile(path: String)
    fun fileExists(path: String): Boolean
    fun getAppDirectory(): String
}

// desktopMain
actual object FileSystem {
    actual fun getAppDirectory(): String {
        return System.getProperty("user.home") + "/.myapp"
    }
    // ...implementations using java.io.File
}
```

### Паттерн 5: expect value class (лёгкие обёртки)

Для типов-обёрток, не несущих аллокации:

```kotlin
// commonMain
expect value class UserId(val value: String) {
    companion object {
        fun generate(): UserId
    }
}

// androidMain
actual value class UserId(val value: String) {
    actual companion object {
        actual fun generate() = UserId(UUID.randomUUID().toString())
    }
}
```

---

## 33.3. Антипаттерны

### Антипаттерн 1: Бизнес-логика в actual

```kotlin
// ❌ Плохо — разная логика на разных платформах
// androidMain
actual fun calculateDiscount(price: Double): Double = price * 0.9

// iosMain
actual fun calculateDiscount(price: Double): Double = price * 0.85
// Это баг! Бизнес-правило должно быть в commonMain.
```

**Правило:** Если логика одинакова — она в `commonMain`. `expect/actual` — только для доступа к платформенным API.

### Антипаттерн 2: expect/actual для строковых констант

```kotlin
// ❌ Плохо — используйте Compose Resources
expect fun appName(): String
// androidMain: actual fun appName() = "MyApp"
// iosMain: actual fun appName() = "MyApp"
```

Строковые константы, которые одинакова на всех платформах — в `commonMain/resources/strings.xml` через Compose Resources.

### Антипаттерн 3: Глубокие иерархии expect-классов

```kotlin
// ❌ Плохо — actual не может менять иерархию наследования
expect open class Base()
expect class Derived() : Base()
// Если на одной платформе Derived нужно наследовать от другого класса — тупик.
```

Ограничение: `actual class` может расширять `expect class`, но не может расширять дополнительные классы сверх тех, что указаны в `expect`. Держите иерархии плоскими — предпочитайте композицию (делегирование) наследованию.

### Антипаттерн 4: Слишком много expect/actual

Если у вас 50+ expect-объявлений — это сигнал, что архитектура перегружена платформенными деталями. Рассмотрите:
- Вынесите платформенный код в отдельный модуль (`:shared:platform`) и зависите от него через интерфейсы.
- Используйте библиотеки, которые уже абстрагируют платформенные различия (Ktor, SQLDelight, multiplatform-settings).

---

## 33.4. expect/actual в тестах

`expect/actual` — ваш друг в тестах. Подменяйте платформенные реализации на фейковые:

```kotlin
// commonTest
actual class PlatformLogger {
    val logs = mutableListOf<String>()
    actual fun d(tag: String, message: String) {
        logs.add("D [$tag] $message")
    }
    actual fun e(tag: String, message: String, throwable: Throwable?) {
        logs.add("E [$tag] $message ${throwable?.message ?: ""}")
    }
    actual fun w(tag: String, message: String) {
        logs.add("W [$tag] $message")
    }
    fun clear() { logs.clear() }
}
```

---

## 33.5. Отладка expect/actual

**Частая ошибка: «Unresolved reference: actual»**
- Убедитесь, что файл в правильном source set (например, `iosMain/kotlin/...`, а не просто `iosMain/`).
- Проверьте, что `iosMain` зарегистрирован в `build.gradle.kts`.

**Частая ошибка: «Actual class does not have corresponding expect declaration»**
- Проверьте полностью квалифицированные имена (package + class name).
- Если используете Gradle module, убедитесь, что common-модуль подключён как dependency.

**Инструмент:** `./gradlew :composeApp:allTests` — запуск тестов на всех платформах найдёт все проблемы с expect/actual.

---

## 33.6. Swift Export и SPM import (Kotlin 2.4)

Начиная с Kotlin 2.4.0, в Kotlin Multiplatform появились две важные возможности для iOS-интеграции:

### Swift Export (Alpha)

Раньше Kotlin/Native генерировал Objective-C framework, и Swift-код взаимодействовал с Kotlin через Objective-C bridging. Это работало, но имело ограничения: имена типов манглись, не было нативных Swift generics, не работали Swift-only фичи (like `some` opaque types).

Swift Export (статус Alpha в Kotlin 2.4) — новая генерация фреймворков напрямую в Swift, минуя Objective-C:

```kotlin
// build.gradle.kts
kotlin {
    iosArm64()
    iosSimulatorArm64()

    targets.withType<org.jetbrains.kotlin.gradle.plugin.mpp.KotlinNativeTarget> {
        binaries.framework {
            baseName = "SharedModule"
            isStatic = true
        }
    }
}
```

Включение Swift Export в `gradle.properties`:
```properties
kotlin.native.binary.objcExport=false
kotlin.native.binary.swiftExport=true
```

> **Статус:** Swift Export — Alpha в Kotlin 2.4.0, стабилизация ожидается в Kotlin 2.5. Для production пока используйте стандартный Objective-C export.

### SPM import — Swift packages как зависимости

Kotlin 2.4 также добавил возможность объявлять Swift packages как зависимости KMP-модуля. Раньше для использования Swift-библиотеки из Kotlin/Native нужно было:
1. Иметь Swift-библиотеку.
2. Создать Objective-C wrapper.
3. Импортировать Objective-C wrapper в Kotlin.

Теперь можно напрямую:

```kotlin
// build.gradle.kts
kotlin {
    iosArm64()
    iosSimulatorArm64()

    cocoapods {
        // Старый способ — через CocoaPods
        pod("SomeLibrary")
    }

    // Новый способ — через Swift Package Manager (Kotlin 2.4+)
    spmDependencies {
        // Объявляем Swift package как зависимость
        swiftPackage("https://github.com/example/SwiftLib.git") {
            version = "1.0.0"
        }
    }
}
```

### Что это меняет для expect/actual

С Swift Export:
- `expect class` теперь может генерировать настоящий Swift-класс (не Objective-C).
- Имена методов не манглись (нет `doSomethingWithArg:`).
- Swift-extensions работают напрямую.
- Асинхронные функции Kotlin (`suspend`) корректно маппятся в `async`/`await` Swift.

```kotlin
// commonMain
expect class PlatformDateFormatter {
    suspend fun format(timestamp: Long): String
}

// iosMain
actual class PlatformDateFormatter {
    actual suspend fun format(timestamp: Long): String {
        // Реализация через NSDateFormatter
    }
}

// Swift side (со Swift Export):
// let formatter = PlatformDateFormatter()
// let formatted = try await formatter.format(timestamp: 1234567890)
// НЕ: formatter.format(timestamp:) с completion handler
```

---

## 33.7. Когда НЕ использовать expect/actual

С развитием KMP-экосистемы многие задачи, ранее требовавшие expect/actual, теперь решаются готовыми библиотеками:

| Задача | Раньше (expect/actual) | Сейчас (готовое решение) |
|--------|----------------------|--------------------------|
| Файлы | `expect class FileSystem` | `multiplatform-file` или `okio` (Multiplatform) |
| Настройки | `expect class Settings` | `multiplatform-settings` |
| Сеть | `expect class HttpClient` | `Ktor` (нативный multiplatform) |
| БД | `expect class Database` | `SQLDelight` (нативный multiplatform) |
| Логирование | `expect class Logger` | `Napier` или `Kermit` |
| Дата/время | `expect class Clock` | `kotlinx-datetime` |
| Сериализация | `expect class Serializer` | `kotlinx.serialization` |

**Правило:** если есть зрелая multiplatform-библиотека для задачи — используйте её, не пишите expect/actual. expect/actual нужен для:
- Платформенно-специфичных UI-интеграций (например, доступ к Camera2 на Android и AVCaptureSession на iOS).
- Кастомных системных API, для которых нет библиотеки.
- Когда вы сознательно хотите тонкую обёртку без зависимости.

---

⬅️ [← Управление состоянием](32-state-management-deep-dive.md) | [К содержанию](README.md) | [KMP gotchas →](34-kmp-gotchas.md) ➡️