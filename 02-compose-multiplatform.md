# Compose Multiplatform

> Compose Multiplatform — фреймворк от JetBrains для создания кроссплатформенных UI-компонентов на Kotlin. Позволяет писать декларативный UI, который можно использовать на Android, iOS, Web (JS и Wasm), Desktop и других платформах.

---

## 2.1. Установка и настройка проекта

1. **Инструменты:** установите IntelliJ IDEA 2026.1+ (или Fleet с поддержкой KMP), JDK 17+ (рекомендуется 21 LTS), Gradle 8.8+ (или используйте Gradle Wrapper).
2. **Создание проекта:** 
   - Самый простой путь — [KMP Web Wizard](https://kmp.jetbrains.com/) — интерактивный онлайн-конструктор, генерирует готовый проект.
   - В IDEA: _New Project → Kotlin Multiplatform → Compose Multiplatform Application_ (нужен плагин Kotlin и Compose Multiplatform).
3. **Структура проекта (стандарт 2026):**
   - `composeApp/` — общий модуль (UI, бизнес-логика)
   - `androidApp/`, `iosApp/` — платформенные точки входа
   - `wasmJsApp/` или `webApp/` — веб-точка входа (опционально)
   - `desktopApp/` — точка входа для десктопа (опционально)

> **Важно:** Начиная с Compose Multiplatform 1.10 (апрель 2026), стандартная структура проекта использует `composeApp` вместо `shared` как имя общего модуля. Это изменение JetBrains сделала для согласования с AGP 9.0 и более чёткого разделения ответственностей модулей.

**Файл `build.gradle.kts` (общий модуль, актуальная конфигурация):**
```kotlin
kotlin {
    androidTarget()

    listOf(
        iosX64(),
        iosArm64(),
        iosSimulatorArm64()
    ).forEach { iosTarget ->
        iosTarget.binaries.framework {
            baseName = "ComposeApp"
            isStatic = true
        }
    }

    // Kotlin/Wasm — компиляция в WebAssembly для веба
    wasmJs {
        browser {
            commonWebpackConfig {
                outputFileName = "composeApp.js"
            }
        }
    }

    // Desktop (JVM)
    jvm("desktop")

    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation(compose.runtime)
                implementation(compose.foundation)
                implementation(compose.material3)
                implementation(compose.components.resources) // Compose Resources
                implementation(compose.components.uiToolingPreview)
            }
        }
        val androidMain by getting {
            dependencies {
                implementation(compose.preview)
            }
        }
        val wasmJsMain by getting
        val iosMain by getting
        val desktopMain by getting {
            dependencies {
                implementation(compose.desktop.currentOs)
            }
        }
    }
}
```

**Ключевое отличие от предыдущих версий:** `js(BOTH)` заменён на `wasmJs()` — это компиляция в WebAssembly, а не в JavaScript. Kotlin/Wasm обеспечивает значительно лучшую производительность (ближе к нативной) по сравнению с Kotlin/JS благодаря прямому выполнению в виртуальной машине браузера без промежуточного JavaScript-слоя. Поддержка Kotlin/JS всё ещё доступна через `js(IR) { browser() }`, но Wasm рекомендуется как основной таргет для новых проектов.

**Плагин Compose Multiplatform (актуально на 30 июня 2026):**
```kotlin
plugins {
    id("org.jetbrains.kotlin.multiplatform") version "2.4.0"
    id("org.jetbrains.kotlin.plugin.compose") version "2.4.0"
    id("org.jetbrains.compose") version "1.11.0"
}
```

> **С Kotlin 2.0+** Compose Compiler plugin выделен в отдельный артефакт `org.jetbrains.kotlin.plugin.compose`. Это позволяет обновлять Compose Compiler независимо от Kotlin. До Kotlin 2.0 использовалась версия Compose Compiler, привязанная к версии Kotlin.

> Всегда проверяйте [таблицу совместимости версий](https://github.com/JetBrains/compose-multiplatform/blob/master/VERSIONING.md#kotlin-compatibility) перед обновлением — Compose Multiplatform привязан к конкретным версиям Kotlin.

---

## 2.2. Декларативный UI

Компоненты Compose описываются как функции с аннотацией `@Composable`, которые описывают дерево UI-элементов. Компилятор Compose преобразует эти функции в эффективное дерево узлов, которое обновляется только при изменении зависимых данных.

**Пример экрана:**
```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Text(
        text = "Hello $name!",
        fontSize = 24.sp,
        modifier = modifier
    )
}
```

**Ключевые принципы:**
- **Композируемые функции** (`@Composable`) — описывают UI как набор вложенных вызовов.
- **Состояние** (`mutableStateOf()`) — управляет динамическими данными и триггерит перекомпоновку.
- **Modifier** — цепочка декораторов для размера, отступов, кликов, скролла и т.д.
- **Layout** — `Column`, `Row`, `Box`, `LazyColumn` для компоновки.

Декларативный подход означает, что вы описываете _что_ должно быть на экране, а не _как_ это отрисовать. Когда данные изменяются, Compose автоматически перерисовывает только те компоненты, которые зависят от изменённых данных (smart recomposition). Это делает код более предсказуемым и удобным для отладки по сравнению с императивным подходом, где вам приходится вручную обновлять каждый элемент при изменении состояния.

**Пример с Material 3 компонентами:**
```kotlin
@Composable
fun TaskCard(task: Task, onToggle: () -> Unit, modifier: Modifier = Modifier) {
    ElevatedCard(modifier = modifier.fillMaxWidth().padding(horizontal = 16.dp)) {
        Row(
            modifier = Modifier.padding(16.dp).fillMaxWidth(),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Checkbox(checked = task.completed, onCheckedChange = { onToggle() })
            Spacer(modifier = Modifier.width(12.dp))
            Text(
                text = task.title,
                textDecoration = if (task.completed) TextDecoration.LineThrough else null
            )
        }
    }
}
```

---

## 2.3. Управление состоянием

Состояние в Compose — это наблюдаемые значения, которые триггерят перекомпоновку при изменении. Compose отслеживает, какие `@Composable`-функции читают какие состояния, и перекомпонует только затронутые функции.

**Пример счётчика:**
```kotlin
@Composable
fun Counter(modifier: Modifier = Modifier) {
    var count by remember { mutableIntStateOf(0) }
    Button(onClick = { count++ }, modifier = modifier) {
        Text("Count: $count")
    }
}
```

- `remember` — сохраняет значение между перекомпоновками (аналог кэша).
- `mutableIntStateOf` — создаёт наблюдаемое значение для примитивного типа (предпочтительнее `mutableStateOf(0)`, т.к. избегает boxing).

Управление состоянием — один из ключевых аспектов Compose. Когда значение состояния изменяется, Compose автоматически инициирует перекомпоновку всех `@Composable`-функций, которые читают это значение. Это гарантирует, что UI всегда отражает актуальное состояние данных. Для более сложных сценариев используйте `StateFlow`, `ViewModel` или библиотеку Decompose.

**Продвинутый пример с ViewModel и StateFlow:**
```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)
    val count: StateFlow<Int> = _count.asStateFlow()

    fun increment() { _count.value++ }
    fun decrement() { _count.value-- }
}

@Composable
fun CounterScreen(viewModel: CounterViewModel = viewModel()) {
    val count by viewModel.count.collectAsStateWithLifecycle()
    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(text = "Count: $count", style = MaterialTheme.typography.headlineLarge)
        Spacer(modifier = Modifier.height(16.dp))
        Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
            Button(onClick = { viewModel.decrement() }) { Text("-") }
            Button(onClick = { viewModel.increment() }) { Text("+") }
        }
    }
}
```

> **Подсказка:** Используйте `collectAsStateWithLifecycle()` вместо `collectAsState()` в Android-приложениях — он автоматически приостанавливает сбор при переходе приложения в фон и возобновляет при возврате, экономя ресурсы.

**State Hoisting (восходящее состояние):**
```kotlin
// Плохо — состояние заперто внутри компонента
@Composable
fun BadCounter() {
    var count by remember { mutableIntStateOf(0) }
    // Нельзя получить/изменить count снаружи
}

// Хорошо — состояние поднято к вызывающему
@Composable
fun GoodCounter(
    count: Int,
    onIncrement: () -> Unit,
    modifier: Modifier = Modifier
) {
    Button(onClick = onIncrement, modifier = modifier) {
        Text("Count: $count")
    }
}
```

State hoisting делает компоненты переиспользуемыми, тестируемыми и предсказуемыми. Родительский компонент управляет состоянием и передаёт его вниз как параметры — это основа однонаправленного потока данных (UDF).

---

## 2.4. Платформенная логика (expect/actual)

Чтобы разделить платформенный и общий код, используйте механизм `expect`/`actual`. В общем модуле вы объявляете `expect`-декларацию (функцию, класс, свойство, объект), а каждый платформенный source set предоставляет свою `actual`-реализацию.

**Общий модуль — `Platform.kt`:**
```kotlin
expect fun getPlatformName(): String

expect class PlatformContext()

@Composable
expect fun getSystemTheme(): ThemeMode
```

**Реализация для Android (`androidMain`):**
```kotlin
actual fun getPlatformName() = "Android ${android.os.Build.VERSION.SDK_INT}"

actual class PlatformContext(val context: android.content.Context)

@Composable
actual fun getSystemTheme(): ThemeMode {
    val configuration = LocalConfiguration.current
    return if (
        configuration.uiMode and android.content.res.Configuration.UI_MODE_NIGHT_MASK
        == android.content.res.Configuration.UI_MODE_NIGHT_YES
    ) ThemeMode.Dark else ThemeMode.Light
}
```

**Реализация для iOS (`iosMain`):**
```kotlin
import platform.UIKit.UIScreen

actual fun getPlatformName() = "iOS"

actual class PlatformContext

@Composable
actual fun getSystemTheme(): ThemeMode {
    // Определяется через UIKit interop
    return ThemeMode.Light // упрощённый пример
}
```

Механизм `expect`/`actual` — мощный инструмент для написания кроссплатформенного кода. Он обеспечивает полную типобезопасность: если вы забыли предоставить `actual`-реализацию для какого-либо таргета, компилятор выдаст ошибку. Это лучше, чем рефлексия или строковые проверки в рантайме, потому что ошибки обнаруживаются на этапе компиляции.

**Когда использовать `expect/actual`:**
- Доступ к платформенным API (камера, файловая система, биометрия).
- Различные реализации алгоритмов для разных платформ.
- Доступ к нативным SDK (Firebase, Google Play Services, Apple frameworks).
- Получение системной информации (размер экрана, тема, локаль).

**Когда НЕ использовать:**
- Разная бизнес-логика на разных платформах — это нарушает принцип единого кода.
- Простые строковые константы — используйте Compose Resources.

---

## 2.5. Compose Resources

Начиная с Compose Multiplatform 1.7.0, доступна единая система ресурсов `compose.components.resources`, которая заменяет платформенные механизмы (strings.xml, .strings, JSON) единым кроссплатформенным API. Ресурсы размещаются в `commonMain/resources/` и автоматически доступны на всех платформах.

**Структура ресурсов:**
```
composeApp/src/commonMain/resources/
├── strings/
│   └── strings.xml          # или values/strings.xml
├── drawable/
│   ├── logo.png
│   └── icon.svg
├── font/
│   └── roboto.ttf
└── files/
    └── config.json
```

**Использование строковых ресурсов:**
```kotlin
import org.jetbrains.compose.resources.stringResource
import mypackage.composeapp.generated.resources.Res
import mypackage.composeapp.generated.strings

@Composable
fun GreetingScreen() {
    Text(text = stringResource(Res.string.app_name))
    Text(text = stringResource(Res.string.greeting, "Alice"))
}
```

**Использование drawable-ресурсов:**
```kotlin
import org.jetbrains.compose.resources.painterResource

@Composable
fun Logo() {
    Image(
        painter = painterResource(Res.drawable.logo),
        contentDescription = "App logo"
    )
}
```

**Использование шрифтов:**
```kotlin
import org.jetbrains.compose.resources.Font
import mypackage.composeapp.generated.resources.Res
import mypackage.composeapp.generated.resources.Roboto

val RobotoFont = Font(Res.font.Roboto)

@Composable
fun ThemedText() {
    Text(
        text = "Custom font",
        fontFamily = FontFamily(RobotoFont)
    )
}
```

> **Важно:** После добавления или изменения ресурсов необходимо запустить задачу `./gradlew generateComposeResourceGenerator` или просто пересобрать проект — Compose Multiplatform сгенерирует типобезопасные классы `Res.string.*`, `Res.drawable.*` и т.д. в директории `build/generated/`.

---

## 2.6. Сборка и запуск

- **Android:** стандартный процесс сборки через Gradle — `./gradlew :androidApp:assembleDebug`.
- **iOS:** откройте `iosApp/iosApp.xcworkspace` в Xcode и запустите на симуляторе или устройстве.
- **Web (Wasm):** `./gradlew :composeApp:wasmJsBrowserDevelopmentRun` — запустит dev-сервер с hot reload.
- **Desktop:** `./gradlew :composeApp:run` — запустит JVM-приложение на десктопе.

**Gradle-задачи для сборки:**
```bash
# Android
./gradlew :androidApp:assembleDebug
./gradlew :androidApp:assembleRelease

# iOS (требует macOS + Xcode)
./gradlew :composeApp:linkDebugFrameworkIosSimulatorArm64
./gradlew :composeApp:linkReleaseFrameworkIosArm64

# Web (Wasm)
./gradlew :composeApp:wasmJsBrowserProductionWebpack    # Production-сборка
./gradlew :composeApp:wasmJsBrowserDevelopmentRun       # Dev-сервер

# Desktop
./gradlew :composeApp:run
./gradlew :composeApp:packageDmg         # macOS
./gradlew :composeApp:packageMsi         # Windows
./gradlew :composeApp:packageDeb         # Linux

# Все тесты
./gradlew :composeApp:allTests
```

Процесс сборки зависит от целевой платформы. Для Android используется стандартный Gradle-плагин, генерирующий APK или AAB. Для iOS Compose Multiplatform генерирует нативный Cocoa Touch Framework, который интегрируется в Xcode-проект через Gradle-плагин. Для Web компилятор Kotlin/Wasm генерирует WebAssembly-бандл, который работает значительно быстрее JavaScript-версии и поддерживается всеми современными браузерами (Chrome 119+, Firefox 120+, Safari 17.4+). Для Desktop компилируется нативное приложение через JVM с поддержкой нативного look-and-feel на каждой платформе.

---

⬅️ [← Основы Kotlin](01-kotlin-basics.md) | [К содержанию](README.md) | [Material Design 3 →](03-material-design-3.md) ➡️