# Платформенная интеграция

> Каждая платформа имеет свои уникальные API и сервисы. В этой части рассматривается интеграция с платформенными возможностями на Android, iOS, Web и Desktop.

---

## 11.1. Android

- **Сервисы Google Play:** Firebase Analytics, Crashlytics.
- **Уведомления:** WorkManager, Firebase Messaging.
- **Камера:** `androidx.camera` API.

Android — наиболее развитая платформа в экосистеме Kotlin Multiplatform. Сервисы Google Play предоставляют широкий набор инструментов: аналитика (Firebase Analytics), отслеживание сбоев (Crashlytics), push-уведомления (Firebase Cloud Messaging), аутентификация и многое другое. WorkManager позволяет планировать фоновые задачи, которые гарантированно выполнятся даже после перезагрузки устройства.

**Пример интеграции Firebase:**
```kotlin
// В androidApp/build.gradle.kts
implementation(platform("com.google.firebase:firebase-bom:32.0.0"))
implementation("com.google.firebase:firebase-analytics")
implementation("com.google.firebase:firebase-crashlytics")
```

---

## 11.2. iOS

- **CoreData** — альтернатива SQLDelight для сложных моделей данных.
- **Camera API** — доступ к камере через Swift interop.
- **Push-уведомления:** UserNotifications framework.

iOS-интеграция в Kotlin Multiplatform реализуется через Kotlin/Native interop с Swift/Objective-C. Вы можете вызывать iOS API напрямую из Kotlin-кода, используя `expect/actual` для абстракции платформенных различий. CoreData — нативный фреймворк Apple для работы с базами данных, который может использоваться как альтернатива SQLDelight, если вам нужно глубокое интегрирование с iOS-экосистемой.

**Пример expect/actual для камеры:**
```kotlin
// Общий модуль
expect class PlatformCamera() {
    fun takePhoto(onResult: (ByteArray) -> Unit)
}

// iOS модуль
actual class PlatformCamera {
    actual fun takePhoto(onResult: (ByteArray) -> Unit) {
        // Использование UIImagePickerController через Swift interop
    }
}
```

---

## 11.3. Web

- **Сборка с webpack:** настройте Gradle для интеграции с веб-инструментами.
- **Оптимизация:** минификация, tree-shaking.
- **Адаптация:** используйте `jsInterop` для работы с JS-библиотеками.

Web-платформа в Compose Multiplatform компилирует Kotlin-код в JavaScript, который работает в браузере. Для сборки используется Gradle с поддержкой webpack. Kotlin/JS поддерживает прямое взаимодействие с JavaScript через `js()`-функцию и `external`-объявления, что позволяет использовать любую JS-библиотеку.

**Пример взаимодействия с JS:**
```kotlin
// Объявление external-функции
external fun alert(message: String)

// Использование
@Composable
fun WebFeature() {
    Button(onClick = { alert("Hello from Kotlin/JS!") }) {
        Text("Show Alert")
    }
}
```

---

## 11.4. Работа с медиа (камера, галерея, файлы)

Кроссплатформенная работа с медиа требует `expect/actual` для доступа к платформенным API. Начиная с 2025 года, библиотеки вроде [Compose Multiplatform Media](https://github.com/nicreboy/Compose-Multiplatform-Media) и [moko-media](https://github.com/icerockdev/moko-media) упрощают этот процесс.

**expect/actual для выбора изображения из галереи:**
```kotlin
// commonMain
expect class ImagePicker() {
    fun pickImage(onResult: (ByteArray?) -> Unit)
}

// androidMain
actual class ImagePicker {
    actual fun pickImage(onResult: (ByteArray?) -> Unit) {
        // Использование ActivityResultLauncher с PickVisualMedia
    }
}

// iosMain  
actual class ImagePicker {
    actual fun pickImage(onResult: (ByteArray?) -> Unit) {
        // Использование PHPickerViewController через Swift interop
    }
}
```

**Разрешения (permissions) — кроссплатформенная абстракция:**
```kotlin
// commonMain
expect fun checkPermission(permission: Permission): Boolean
expect fun requestPermission(permission: Permission, onResult: (Boolean) -> Unit)

enum class Permission { Camera, Gallery, Notifications, Location }
```

---

## 11.5. Фоновые задачи (Background Work)

Фоновые задачи (синхронизация, загрузка, уведомления) зависят от платформы. Для кроссплатформенного подхода используйте `expect/actual` или библиотеки-обёртки.

**Android:**
- **WorkManager** — гарантированное выполнение (даже после перезагрузки), для периодических задач.
- **Coroutine + Service** — для задач привязанных к жизни приложения.

**iOS:**
- **BGTaskScheduler** — фоновые задачи с лимитом ~30 секунд.
- **Background Fetch** — периодическая загрузка данных (контролируется ОС).

**Кроссплатформенная абстракция:**
```kotlin
// commonMain
expect class BackgroundScheduler() {
    fun schedulePeriodic(taskId: String, intervalMs: Long, task: () -> Unit)
    fun scheduleOneTime(taskId: String, delayMs: Long, task: () -> Unit)
    fun cancel(taskId: String)
}

// androidMain
actual class BackgroundScheduler {
    actual fun schedulePeriodic(taskId: String, intervalMs: Long, task: () -> Unit) {
        val request = PeriodicWorkRequestBuilder<SyncWorker>(
            repeatInterval = intervalMs, TimeUnit.MILLISECONDS
        ).build()
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(
            taskId, ExistingPeriodicWorkPolicy.KEEP, request
        )
    }
    // ...
}

// iosMain
actual class BackgroundScheduler {
    actual fun schedulePeriodic(taskId: String, intervalMs: Long, task: () -> Unit) {
        // BGAppRefreshTask через Swift interop
    }
    // ...
}
```

> **Важно:** iOS агрессивно ограничивает фоновые задачи. Не полагайтесь на точное время выполнения — ОС может задержать или отменить задачу для экономии батареи. Критичные операции (платежи, важные уведомления) лучше выполнять через push-уведомления с сервера.

---

## 11.6. Desktop (Compose Desktop)

- Установите `compose-desktop` плагин.
- Настройте `build.gradle`:
  ```kotlin
  jvm("desktop") {
      withJava()
  }
  ```
- Запустите приложение как обычное JVM-приложение.

Compose Desktop позволяет создавать десктопные приложения для Windows, macOS и Linux, используя тот же декларативный подход, что и для мобильных платформ. Приложения компилируются в JVM-байткод и запускаются через JRE. Для дистрибуции можно создать нативные установщики (MSI, DMG, DEB) через `compose.desktop.application`.

**Пример точки входа:**
```kotlin
fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        title = "My Desktop App"
    ) {
        MyAppTheme {
            TasksScreen(TasksViewModel())
        }
    }
}
```

---

⬅️ [← Производительность](10-performance.md) | [К содержанию](README.md) | [Безопасность →](12-security.md) ➡️
