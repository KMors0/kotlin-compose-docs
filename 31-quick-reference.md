# Шпаргалка: Quick Reference

> Быстрый справочник по ключевым API, командам Gradle и распространённым паттернам Compose Multiplatform. Не заменяет основную документацию — используйте как «one-pager» для быстрого поиска.

---

## Структура проекта

```
composeApp/
├── src/
│   ├── commonMain/
│   │   ├── kotlin/          # Общий код (UI, логика)
│   │   └── resources/       # Compose Resources (строки, картинки, шрифты)
│   ├── androidMain/         # Android-specific код
│   ├── iosMain/             # iOS-specific код (Kotlin/Native)
│   ├── desktopMain/         # Desktop (JVM) код
│   └── wasmJsMain/          # Web (Wasm) код
├── build.gradle.kts
androidApp/
├── src/main/                # Android entry point
iosApp/
├── iosApp.xcworkspace       # Xcode проект
└── Podfile                  # CocoaPods (если используется)
```

---

## Ключевые зависимости (build.gradle.kts)

```kotlin
plugins {
    id("org.jetbrains.kotlin.multiplatform") version "2.1.0"
    id("org.jetbrains.compose") version "1.7.3"
    id("com.android.application") version "8.7.0"
}

kotlin {
    androidTarget()
    listOf(iosX64(), iosArm64(), iosSimulatorArm64()).forEach { it.binaries.framework { isStatic = true } }
    wasmJs { browser { commonWebpackConfig { outputFileName = "composeApp.js" } } }
    jvm("desktop")

    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation(compose.runtime)
                implementation(compose.foundation)
                implementation(compose.material3)
                implementation(compose.components.resources)
                implementation(compose.components.uiToolingPreview)
                // Сеть
                implementation("io.ktor:ktor-client-core:3.0.0")
                implementation("io.ktor:ktor-client-content-negotiation:3.0.0")
                implementation("io.ktor:ktor-serialization-kotlinx-json:3.0.0")
                // Сериализация
                implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.0")
                // DI
                implementation("io.insert-koin:koin-core:4.0.0")
                implementation("io.insert-koin:koin-compose:4.0.0")
                // Корутины
                implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")
            }
        }
        val androidMain by getting {
            dependencies { implementation(compose.preview) }
        }
        val desktopMain by getting {
            dependencies { implementation(compose.desktop.currentOs) }
        }
    }
}
```

---

## Gradle-команды

```bash
# Сборка
./gradlew :androidApp:assembleDebug          # Android APK (debug)
./gradlew :androidApp:assembleRelease          # Android APK (release)
./gradlew :composeApp:linkDebugFrameworkIosSimulatorArm64  # iOS framework
./gradlew :composeApp:wasmJsBrowserDevelopmentRun          # Web dev-сервер
./gradlew :composeApp:wasmJsBrowserProductionWebpack       # Web production
./gradlew :composeApp:run                      # Desktop
./gradlew :composeApp:packageDmg               # macOS .dmg
./gradlew :composeApp:packageMsi               # Windows .msi

# Тесты
./gradlew :composeApp:allTests                 # Все платформы
./gradlew :composeApp:desktopTest               # JVM/Desktop
./gradlew :composeApp:androidUnitTest           # Android
./gradlew :composeApp:iosSimulatorArm64Test     # iOS (требует macOS)

# Ресурсы
./gradlew generateCommonMainComposeResourceGenerator  # Перегенерация Res.*

# Очистка
./gradlew clean                                 # Очистить build/
./gradlew :composeApp:clean                     # Только composeApp
```

---

## State Management Cheat Sheet

```kotlin
// Простое состояние
var count by remember { mutableIntStateOf(0) }

// Состояние с ViewModel
val state by viewModel.uiState.collectAsStateWithLifecycle()

// Derived state (пересчитывается только при реальном изменении)
val filteredTasks by remember {
    derivedStateOf { tasks.filter { it.completed } }
}

// rememberSaveable — сохраняется при повороте экрана
var text by rememberSaveable { mutableStateOf("") }

// StateFlow в ViewModel
class MyViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
}
```

---

## Compose UI Cheat Sheet

```kotlin
// Лейаут
Column(verticalArrangement = Arrangement.spacedBy(8.dp)) { /* ... */ }
Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) { /* ... */ }
Box(contentAlignment = Alignment.Center) { /* ... */ }

// Списки
LazyColumn { items(items, key = { it.id }) { item -> ItemRow(item) } }
LazyRow { items(items) { item -> ItemCard(item) } }

// Модификаторы (цепочка)
Modifier
    .fillMaxWidth()
    .padding(16.dp)
    .clickable { /* onClick */ }
    .background(MaterialTheme.colorScheme.surface)

// Modifier для карточек
Modifier
    .fillMaxWidth()
    .padding(horizontal = 16.dp, vertical = 8.dp)
```

---

## expect/actual Pattern

```kotlin
// commonMain
expect fun getPlatformName(): String
expect class PlatformContext()

// androidMain
actual fun getPlatformName() = "Android ${android.os.Build.VERSION.SDK_INT}"
actual class PlatformContext(val context: android.content.Context)

// iosMain
actual fun getPlatformName() = "iOS"
actual class PlatformContext

// desktopMain
actual fun getPlatformName() = "Desktop (${System.getProperty("os.name")})"
actual class PlatformContext
```

---

## Compose Resources

```kotlin
import org.jetbrains.compose.resources.*
import mypackage.composeapp.generated.resources.*

// Строки
Text(stringResource(Res.string.app_name))
Text(stringResource(Res.string.greeting, "Alice"))

// Плюрализация
Text(stringResource(Res.plurals.task_count, count, count))

// Drawable
Image(painter = painterResource(Res.drawable.logo), contentDescription = "Logo")

// Шрифты
FontFamily(Font(Res.font.Roboto))

// Вне @Composable
suspend fun getTitle() = getString(Res.string.app_name)
```

---

## Ключевые библиотеки KMP

| Категория | Библиотека | Примечание |
|-----------|-----------|------------|
| Сеть | Ktor 3.0 | HTTP-клиент, WebSocket, SSE |
| Сериализация | kotlinx.serialization | Без рефлексии |
| БД | SQLDelight 2.0 | Типобезопасный SQL |
| Настройки | multiplatform-settings | Альтернатива SharedPreferences |
| DI | Koin 4.0 | Лёгкий, без кодогенерации |
| Навигация | Decompose / Voyager | Альтернативы navigation-compose |
| Изображения | Coil 3 / Compose ImageLoader | Асинхронная загрузка |
| Локализация | Compose Resources / Lyricist | Типобезопасные строки |
| Дата/время | kotlinx-datetime | Кроссплатформенный java.time |
| Тесты | kotlin-test + Mockative | Мультиплатформенные |

---

## Версионная совместимость

> Всегда проверяйте актуальную таблицу: https://github.com/JetBrains/compose-multiplatform/blob/master/VERSIONING.md

| Compose MP | Kotlin | Gradle | JDK |
|------------|--------|--------|-----|
| 1.7.x | 2.0.21 | 8.9+ | 17+ |
| 1.8.x | 2.1.0 | 8.10+ | 17+ |
| 1.9.x (beta) | 2.1.10 | 8.11+ | 17+ |

---

## Полезные ссылки

- [Compose Multiplatform Docs](https://www.jetbrains.com/lp/compose-multiplatform/)
- [Kotlin Multiplatform Docs](https://kotlinlang.org/docs/multiplatform.html)
- [Material Design 3](https://m3.material.io/)
- [Compose Resources](https://www.jetbrains.com/help/compose-multiplatform/compose-resources.html)
- [Kotlin/Wasm](https://kotlinlang.org/docs/wasm-overview.html)
- [SQLDelight](https://cashapp.github.io/sqldelight/)
- [Ktor](https://ktor.io/docs/)
- [Koin](https://insert-koin.io/docs/)
- [Decompose](https://github.com/arkivanov/Decompose)

---

⬅️ [← Миграция на KMP](30-migration-guide.md) | [К содержанию](README.md) | [Accessibility →](28-accessibility.md) ➡️
