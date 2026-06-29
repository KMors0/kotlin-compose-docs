# Отладка и устранение ошибок

> Даже в хорошо спроектированном приложении возникают ошибки. В этой части рассматриваются типичные проблемы кроссплатформенной разработки, инструменты логирования и подходы к отладке на разных платформах.

---

## 15.1. Типичные проблемы

- **Конфликты зависимостей:** проверьте `gradle.dependencies` для транзитивных зависимостей.
- **Ошибки сборки:** убедитесь, что версии Kotlin и Compose совместимы.
- **Несовместимость платформ:** используйте условную компиляцию (`iosApplication {}`, `androidApplication {}`).

Конфликты зависимостей — одна из самых частых проблем в Kotlin Multiplatform. Разные библиотеки могут транзитивно подтягивать разные версии одной и той же зависимости. Используйте `./gradlew dependencies` для анализа дерева зависимостей и `resolutionStrategy` для принудительного использования конкретной версии.

**Пример разрешения конфликтов:**
```kotlin
configurations.all {
    resolutionStrategy {
        force("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
    }
}
```

Совместимость версий Kotlin и Compose также является частым источником проблем. Compose Multiplatform привязан к конкретным версиям Kotlin, и использование несовместимой версии приведёт к ошибкам компиляции. Всегда проверяйте [таблицу совместимости](https://github.com/JetBrains/compose-multiplatform#versions) перед обновлением.

---

## 15.2. Логирование

- **slf4j** — кроссплатформенное логирование.
- **Timber** — для Android (красивые логи).
- **Платформа-специфичные API:** `NSLog` (iOS), `console.log` (Web).

**Пример Timber:**
```kotlin
Timber.plant(Timber.DebugTree())
Timber.d("Debug message")
```

Для кроссплатформенного логирования создайте абстракцию через `expect/actual`:

```kotlin
// Общий модуль
expect object AppLogger {
    fun d(tag: String, message: String)
    fun e(tag: String, message: String, throwable: Throwable? = null)
}

// Android
actual object AppLogger {
    actual fun d(tag: String, message: String) = Timber.tag(tag).d(message)
    actual fun e(tag: String, message: String, throwable: Throwable?) =
        Timber.tag(tag).e(throwable, message)
}

// iOS
actual object AppLogger {
    actual fun d(tag: String, message: String) = NSLog("[$tag] $message")
    actual fun e(tag: String, message: String, throwable: Throwable?) =
        NSLog("[$tag] ERROR: $message ${throwable?.message ?: ""}")
}
```

Такой подход позволяет использовать единый API логирования в общем коде, при этом на каждой платформе будет использоваться наиболее подходящий инструмент.

---

## 15.3. Отладка на iOS

- Откройте `.xcworkspace` в Xcode.
- Настройте символы в IDEA: `File → Sync Project with Gradle Files`.
- Используйте точки останова в общем коде — они будут работать в Xcode.

Отладка iOS-части кроссплатформенного приложения требует особого подхода. Kotlin/Native компилирует общий код в нативный бинарник, поэтому стандартные инструменты отладки Kotlin не работают. Вместо этого используйте Xcode Debugger — он поддерживает точки останова в сгенерированном коде и показывает корректные символы, если включена отладочная информация.

**Советы по отладке на iOS:**
- Включите `debuggable = true` в Gradle-конфигурации для debug-сборок.
- Используйте `println()` для быстрого логирования — вывод появится в консоли Xcode.
- Для сложных проблем используйте LLDB-команды прямо в консоли Xcode.
- Проверяйте краш-логи в Organizer → Crashes.

---

⬅️ [← Архитектурные паттерны](14-architecture.md) | [К содержанию](README.md) | [Примеры приложений →](16-real-examples.md) ➡️
