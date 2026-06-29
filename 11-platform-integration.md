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

## 11.6. Работа с нативным кодом (JNI, C-interop)

Иногда Kotlin-кода недостаточно — нужно вызвать функции из C/C++ библиотеки (криптография, обработка сигналов, ML-инференс, legacy-код). В KMP-стеке есть три пути работы с нативным кодом, в зависимости от платформы.

### Когда нужен нативный код

| Сценарий | Пример |
|----------|--------|
| **Криптография** | OpenSSL, BoringSSL, libsodium для нестандартных алгоритмов |
| **Обработка медиа** | FFmpeg, libvpx для видео/аудио кодирования |
| **ML/ИИ** | TensorFlow Lite, ONNX Runtime, libtorch |
| **Высокопроизводительные вычисления** | BLAS/LAPACK для матричных операций, FFT |
| **Legacy C-библиотеки** | корпоративные SDK, закрытые проприетарные движки |
| **Доступ к низкоуровневым API** | HAL на Android, IOKit на macOS |

### Три пути работы с нативным кодом

| Путь | Где работает | Описание |
|------|---------------|----------|
| **JNI (Java Native Interface)** | Только JVM/Android | Вызов C/C++ из Java/Kotlin через `native`-методы и `.so` библиотеки |
| **JNA (Java Native Access)** | Только JVM/Android | Доступ к нативным библиотекам без написания JNI-кода — полностью на Java |
| **Kotlin/Native cinterop** | Kotlin/Native (iOS, macOS, Linux, Wasm) | Прямой вызов C-функций через `.def`-файлы, генерация Kotlin-биндингов |

### JNI — для Android и JVM

JNI — классический механизм вызова C/C++ из Java/Kotlin. Требует написания нативного кода на C/C++ и компиляции в `.so` для каждой архитектуры (arm64-v8a, armeabi-v7a, x86_64).

#### Шаг 1: объявление `external`-функции в Kotlin

```kotlin
// androidMain или jvmMain
class NativeCrypto {
    // Объявление native-метода — реализация в .so библиотеке
    external fun hashSha256(data: ByteArray): ByteArray
    external fun encryptAes(plaintext: ByteArray, key: ByteArray): ByteArray

    companion object {
        init {
            // Загрузка нативной библиотеки (libnative-lib.so на Android, native-lib.dll на Windows и т.д.)
            System.loadLibrary("native-lib")
        }
    }
}
```

#### Шаг 2: реализация на C/C++

`native-lib.cpp`:
```cpp
#include <jni.h>
#include <string>
#include <openssl/sha.h>

extern "C" JNIEXPORT jbyteArray JNICALL
Java_com_example_app_NativeCrypto_hashSha256(
    JNIEnv* env,
    jobject /* this */,
    jbyteArray data
) {
    // Получаем данные из Java-массива
    jbyte* bytes = env->GetByteArrayElements(data, nullptr);
    jsize length = env->GetArrayLength(data);

    // Вычисляем SHA-256 через OpenSSL
    unsigned char hash[SHA256_DIGEST_LENGTH];
    SHA256(reinterpret_cast<const unsigned char*>(bytes), length, hash);

    // Освобождаем Java-массив
    env->ReleaseByteArrayElements(data, bytes, JNI_ABORT);

    // Возвращаем результат как Java-массив
    jbyteArray result = env->NewByteArray(SHA256_DIGEST_LENGTH);
    env->SetByteArrayRegion(result, 0, SHA256_DIGEST_LENGTH, reinterpret_cast<jbyte*>(hash));
    return result;
}
```

#### Шаг 3: сборка через CMake

`androidApp/build.gradle.kts`:
```kotlin
android {
    defaultConfig {
        ndk {
            abiFilters += listOf("arm64-v8a", "x86_64")  // architectures to build
        }
        externalNativeBuild {
            cmake {
                cppFlags += "-std=c++17"
                arguments += "-DANDROID_STL=c++_shared"
            }
        }
    }
    externalNativeBuild {
        cmake {
            path = file("src/main/cpp/CMakeLists.txt")
            version = "3.22.1"
        }
    }
}
```

`src/main/cpp/CMakeLists.txt`:
```cmake
cmake_minimum_required(VERSION 3.22.1)
project("native-lib")

# Добавляем OpenSSL (через prebuilt или FetchContent)
find_package(OpenSSL REQUIRED)

add_library(native-lib SHARED native-lib.cpp)
target_link_libraries(native-lib OpenSSL::SSL OpenSSL::Crypto log)
```

### JNA — упрощённый доступ без JNI-кода

[JNA (Java Native Access)](https://github.com/java-native-access/jna) — библиотека, позволяющая вызывать C-функции из Java/Kotlin **без написания JNI-обёрток**. JNA сама генерирует вызовы через reflection-like API на основе интерфейса.

```kotlin
// build.gradle.kts
dependencies {
    implementation("net.java.dev.jna:jna:5.14.0")
}

// Kotlin — объявление интерфейса с C-функциями
import com.sun.jna.Library
import com.sun.jna.Native
import com.sun.jna.Pointer

interface CryptoLib : Library {
    fun SHA256(data: Pointer, length: Int, hash: Pointer): Int
    fun AES_encrypt(plaintext: Pointer, len: Int, key: Pointer, output: Pointer): Int
}

// Загрузка системной библиотеки (libcrypto.so на Linux/Android, libcrypto.dylib на macOS)
val cryptoLib: CryptoLib = Native.load("crypto", CryptoLib::class.java)

// Использование
fun hashSha256(data: ByteArray): ByteArray {
    val dataPtr = Memory(data.size.toLong())
    dataPtr.write(0, data, 0, data.size)
    val hashPtr = Memory(32)  // SHA-256 = 32 байта

    cryptoLib.SHA256(dataPtr, data.size, hashPtr)

    return hashPtr.getByteArray(0, 32)
}
```

**Сравнение JNI vs JNA:**

| Критерий | JNI | JNA |
|----------|-----|-----|
| **Производительность** | Высокая (direct call) | Низкая на старте (~100x медленнее первого вызова), затем приемлемая |
| **Объём кода** | Нужно писать C/C++ + заголовки | Только Kotlin/Java-интерфейс |
| **Сборка** | Нужен CMake/NDK | Просто gradle-dependency |
| **Размер приложения** | + размер .so | + ~2 МБ JNA-runtime |
| **Поддержка архитектур** | Нужно собирать под все ABI | Автоматически |

> **Когда JNA предпочтительнее:** прототипирование, доступ к простым C-функциям, когда не хочется настраивать NDK-сборку. Для production-кода с высокой нагрузкой — JNI.

### Kotlin/Native cinterop — для iOS, macOS, Linux, Wasm

На платформах Kotlin/Native (iOS, macOS, Linux, Wasm) JNI не работает — нет JVM. Вместо этого используется **cinterop** — инструмент Kotlin/Native, генерирующий Kotlin-биндинги из C-заголовков.

#### Шаг 1: создание `.def`-файла

`composeApp/src/nativeInterop/cinterop/crypto.def`:
```def
headers = openssl/sha.h openssl/aes.h
headerFilter = openssl/*
linkerOpts = -lcrypto
```

#### Шаг 2: подключение в `build.gradle.kts`

```kotlin
kotlin {
    iosArm64()
    iosSimulatorArm64()

    cocoapods {
        // Альтернативный путь — через CocoaPods (если библиотека есть в Pods)
        pod("OpenSSL") {
            version = "1.1.1"
        }
    }

    val cinteropCrypto by cinterops.creating {
        defFile("src/nativeInterop/cinterop/crypto.def")
        packageName("com.example.crypto")
    }

    iosArm64.binaries.framework {
        baseName = "ComposeApp"
        isStatic = true
    }
}
```

#### Шаг 3: использование из Kotlin

```kotlin
// iosMain
import com.example.crypto.openssl.*
import kotlinx.cinterop.*

actual class NativeCrypto {
    actual fun hashSha256(data: ByteArray): ByteArray {
        val hash = UByteArray(SHA256_DIGEST_LENGTH)

        data.usePinned { pinned ->
            hash.usePinned { hashPinned ->
                SHA256(
                    pinned.addressOf(0).reinterpret(),
                    data.size,
                    hashPinned.addressOf(0).reinterpret()
                )
            }
        }

        return hash.toByteArray()
    }
}
```

> **`usePinned` — критично для производительности:** Kotlin/Native не позволяет напрямую работать с указателями на массивы (для safety). `usePinned` фиксирует массив в памяти на время блока и даёт доступ к указателю. После выхода из блока массив может быть перемещён GC.

### expect/actual абстракция для кроссплатформенного нативного кода

Типичный паттерн — абстрагировать нативные вызовы через `expect`/`actual`, чтобы общий код не знал о реализации:

```kotlin
// commonMain
expect class NativeCrypto() {
    fun hashSha256(data: ByteArray): ByteArray
    fun encryptAes(plaintext: ByteArray, key: ByteArray): ByteArray
}

// androidMain — JNI
actual class NativeCrypto {
    actual fun hashSha256(data: ByteArray): ByteArray {
        return jniHashSha256(data)  // JNI-вызов
    }
    actual fun encryptAes(plaintext: ByteArray, key: ByteArray): ByteArray {
        return jniEncryptAes(plaintext, key)
    }

    private external fun jniHashSha256(data: ByteArray): ByteArray
    private external fun jniEncryptAes(plaintext: ByteArray, key: ByteArray): ByteArray

    companion object {
        init { System.loadLibrary("native-lib") }
    }
}

// iosMain — Kotlin/Native cinterop
actual class NativeCrypto {
    actual fun hashSha256(data: ByteArray): ByteArray {
        // Реализация через cinterop, как в примере выше
    }
    actual fun encryptAes(plaintext: ByteArray, key: ByteArray): ByteArray {
        // ...
    }
}

// wasmJsMain — заглушка или Web Crypto API
actual class NativeCrypto {
    actual fun hashSha256(data: ByteArray): ByteArray {
        // Web Crypto API — crypto.subtle.digest("SHA-256", data)
        // или kmp-wasm-js-interop обёртка
        TODO("Implement through Web Crypto API")
    }
    actual fun encryptAes(plaintext: ByteArray, key: ByteArray): ByteArray {
        TODO("Implement through Web Crypto API")
    }
}
```

### Готовые библиотеки вместо самостоятельной работы с JNI

Прежде чем писать JNI-обёртку, проверьте, нет ли готового KMP-решения:

| Задача | Готовая библиотека | Зачем не писать свой JNI |
|--------|-------------------|--------------------------|
| **SHA/AES/RSA** | `kotlin-multiplatform-libsodium` или `AES-KMP` | Уже собрано под все платформы |
| **SQLite** | `SQLDelight` | Не нужен JNI — NativeSqliteDriver уже включён |
| **HTTP/3** | `Ktor + Cronet` (Android) или `Ktor + Darwin` (iOS) | Cronet собран Google, Darwin — Apple |
| **TLS** | Встроено в OkHttp/Darwin engine | Не нужно возиться с OpenSSL |
| **Image processing** | `multiplatform-scoped-image-loader` | Native декодеры под капотом |
| **Audio/Video** | `media-lib-extractor`, `ExoPlayer` (Android), `AVFoundation` (iOS) | Лучше использовать platform-specific |

> **Главное правило:** JNI/cinterop — это инструменты «последней мили». Если есть готовая библиотека — используйте её. Свои JNI-обёртки требуются только для проприетарных SDK или уникальных алгоритмов.

### Предостережения

1. **Безопасность памяти:** ошибки в C/C++ коде (use-after-free, buffer overflow) приводят к крашам всего процесса, а не отдельных исключений. JVM не спасает — `SIGSEGV` в JNI-коде роняет Android-приложение.

2. **App size:** каждая ABI добавляет ~1-3 МБ `.so`-библиотеки. Если включить arm64-v8a + armeabi-v7a + x86_64 — это x3 размера. Используйте App Bundle (AAB) — Google Play сам раздаёт нужный APK под устройство.

3. **Kotlin/Native constraints:** на iOS GC работает иначе, чем JVM. Передача Kotlin-объектов в C и обратно требует `StableRef` для удержания ссылок — легко создать утечку памяти.

4. **Debugging:** отладка JNI-кода требует LLDB/GDB, обычный Kotlin-debugger не работает. Используйте Android Studio's Native Debug mode или Xcode для iOS.

---

## 11.7. Desktop (Compose Desktop)

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
