# Оптимизация размера приложения

> Compose Multiplatform приложения, особенно для iOS, имеют ощутимый начальный размер (~15–25 МБ для минимального приложения). Для comparison: нативное SwiftUI-приложение может весить < 5 МБ. Этот раздел описывает реальные техники уменьшения размера, применимые к KMP-проектам.

---

## 37.1. Профилирование размера

Прежде чем оптимизировать — измерьте.

**Android:**
```bash
# Размер APK/AAB по компонентам
./gradlew :androidApp:assembleRelease
apkanalyzer --human-readable androidApp/build/outputs/apk/release/app-release.apk
```

`apkanalyzer` покажет: общий размер, размер по типам файлов (DEX, native libs, assets, resources), и какие классы/методы занимают больше всего места.

**iOS:**
```bash
# Размер IPA
xcodebuild -showBuildSettings -project iosApp/iosApp.xcodeproj | grep PRODUCT_BUNDLE_IDENTIFIER
# В Xcode: Products → Show in Finder → Get Info
# Или через Instruments → Allocations при запуске на устройстве
```

**Web (Wasm):**
```bash
./gradlew :composeApp:wasmJsBrowserProductionWebpack
ls -lh composeApp/build/dist/wasmJs/productionExecutable/
# В Chrome DevTools: Network tab → размер wasm-бандла
```

---

## 37.2. R8 / ProGuard для Android

R8 — инструмент обфускации и сжатия кода для Android. Он удаляет неиспользуемый код (tree-shaking), обфусцирует имена и сжимает строковые константы.

**Включение R8 (по умолчанию включён для release):**
```kotlin
// androidApp/build.gradle.kts
android {
    buildTypes {
        release {
            isMinifyEnabled = true   // R8 code shrinking + obfuscation
            isShrinkResources = true  // Удаление неиспользуемых ресурсов
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

**Частые proguard-правила для KMP:**
```proguard
# kotlinx.serialization
-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.AnnotationsKt
-keepclassmembers class kotlinx.serialization.json.** { *** Companion; }

# Ktor
-dontnote io.ktor.**
-keep class io.ktor.serialization.kotlinx.** { *; }

# Koin
-keep class org.koin.** { *; }

# SQLDelight
-keep class com.squareup.sqldelight.** { *; }
```

Типичное сокращение: R8 уменьшает размер APK на **30–60%**.

---

## 37.3. Оптимизация Kotlin/Native binaries (iOS)

Kotlin/Native компилирует в нативный машинный код. Размер binary можно уменьшить:

**1. Компиляция в статический фреймворк (isStatic = true):**
```kotlin
// Уже рекомендуется в стандартной настройке
iosArm64().binaries.framework {
    baseName = "ComposeApp"
    isStatic = true  // Статическая линковка — удаляет динамические символы
}
```

**2. Оптимизация компилятора:**
```kotlin
kotlin {
    iosArm64 {
        binaries.framework {
            binaryOptions["-Osize"] = null  // Оптимизация по размеру (вместо -O2)
        }
    }
}
```

**3. Исключение отладочной информации:**
```kotlin
// В release-конфигурации
binaries.framework {
    isStatic = true
    optimizationMode = org.jetbrains.kotlin.gradle.plugin.mpp.NativeBuildType.RELEASE
}
```

---

## 37.4. Tree-shaking Web (Wasm)

Для Web-сборки основной размер занимает Skia (графический движок) и Kotlin stdlib.

**Оптимизации в build.gradle.kts:**
```kotlin
wasmJs {
    browser {
        commonWebpackConfig {
            outputFileName = "composeApp.js"
            // Webpack tree-shaking (по умолчанию включён для production)
            devServer = null // Отключить dev-сервер в release
        }
        testTask {
            useKarma {
                useChromeHeadless()
            }
        }
    }
}
```

**Дополнительные приёмы:**
- Минимизируйте количество зависимостей в `commonMain` — каждая библиотека увеличивает wasm-бандл.
- Используйте `js(IR) { browser() }` для legacy-js targets только если нужна поддержка старых браузеров.
- Lazy-загрузка тяжёлых модулей через `@JsModule`.

---

## 37.5. Ресурсы и изображения

**1. WebP вместо PNG/JPEG:** WebP обеспечивает на 25–35% меньший размер при том же качестве.

**2. Vector drawable (SVG):** Compose Multiplatform поддерживает SVG через Compose Resources. Векторные изображения масштабируются без потери качества и занимают мало места.

**3. Удаление неиспользуемых ресурсов:**
```kotlin
// build.gradle.kts (android)
android {
    buildTypes {
        release {
            isShrinkResources = true
        }
    }
}
```

---

## 37.6. Базовые показатели (для ориентира)

| Платформа | Минимальное приложение | С R8/shrinking | С оптимизацией ресурсов |
|-----------|---------------------|----------------|------------------------|
| Android APK | 12–18 МБ | 5–8 МБ | 4–6 МБ |
| iOS IPA | 15–25 МБ | 12–18 МБ | 10–15 МБ |
| Web (Wasm) | 3–5 МБ (wasm + js) | 2–3 МБ | 2–3 МБ |
| Desktop (JVM) | 30–50 МБ (с JRE) | 20–35 МБ | jlink: 15–25 МБ |

> Размеры приблизительны и зависят от количества зависимостей и используемых платформенных SDK. Цель — не «минимум байт», а «разумный размер без потери функциональности».

---

⬅️ [← Адаптивные лейауты](31-adaptive-layouts.md) | [К содержанию](README.md) | [Шпаргалка →](31-quick-reference.md) ➡️
