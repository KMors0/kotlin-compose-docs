# Глубокое погружение в Compose Multiplatform

> Детальное рассмотрение архитектуры Compose Multiplatform: декларативная парадигма, матрица поддержки платформ на 30 июня 2026 года, организация модулей, навигация и управление состоянием. Все версии и API актуальны на момент написания — Compose Multiplatform 1.11.0 stable, Kotlin 2.4.0.

---

## 21.1. Декларативная парадигма UI

Compose основан на декларативной парадигме: разработчик описывает, *как должен выглядеть* UI при определённом состоянии, а фреймворк берёт на себя ответственность за обновление экрана при изменении состояния. Это принципиально отличается от императивного подхода (Android View system, UIKit), где разработчик вручную управляет обновлениями UI-дерева.

```kotlin
@Composable
fun Greeting(name: String) {
    var expanded by remember { mutableStateOf(false) }

    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
            .clickable { expanded = !expanded }
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(
                text = "Hello, $name!",
                style = MaterialTheme.typography.headlineMedium
            )
            if (expanded) {
                Text(
                    text = "Compose Multiplatform 1.11 позволяет запускать этот код на Android, iOS, Desktop и Web (Wasm).",
                    style = MaterialTheme.typography.bodyMedium
                )
            }
        }
    }
}
```

### Ключевые концепции Compose

| Концепция | Описание |
|-----------|----------|
| **Composable-функции** | Функции, аннотированные `@Composable`, описывают часть UI. Они не возвращают UI-компоненты, а отправляют описания в runtime Compose |
| **Состояние (State)** | Объекты, обёрнутые в `State<T>`, изменение которых вызывает рекомпозицию затронутых частей UI |
| **Модификаторы (Modifiers)** | Цепочки декораторов, определяющих визуальные свойства компонента — размер, отступы, кликабельность, скругление |
| **Рекомпозиция** | Процесс обновления только тех частей UI-дерева, чьи состояния изменились (smart recomposition) |
| **remember** | Механизм кэширования объектов между рекомпозициями для предотвращения их пересоздания |
| **Snapshot** | Транзакционная система отслеживания изменений состояния; обеспечивает атомарность и предсказуемость |

Compose Multiplatform обеспечивает **полный паритет API** с Jetpack Compose для Android. Все layout-контейнеры (`Row`, `Column`, `Box`, `LazyColumn`, `LazyRow`), анимации (`animateContentSize`, `AnimatedVisibility`, `updateTransition`, `SharedTransitionScope`), жесты (`detectTapGestures`, `scrollable`, `draggable`) и системные API работают идентично на всех платформах. Расхождения есть только в платформенно-специфичных интеграциях (см. раздел 21.6).

---

## 21.2. Матрица поддержки платформ (на 30 июня 2026)

Compose Multiplatform на середину 2026 года поддерживает следующие платформы:

| Платформа | Статус | Бэкенд рендеринга | Ключевые особенности |
|-----------|--------|-------------------|---------------------|
| **Android** | Stable (полный паритет с Jetpack Compose 1.11.2) | Android Canvas | Полный доступ к Android API, Material 3, динамические цвета |
| **iOS** | Stable (с 1.8.0, май 2025; зрелость в 1.11.0) | Metal / Core Animation через Skia (skiko M138) | VoiceOver, AssistiveTouch, Full Keyboard Access, 120 Гц |
| **Desktop (JVM)** | Stable | Skia (OpenGL) | JDK 11+, нативное оконение, поддержка menu bar |
| **Web (Kotlin/Wasm)** | Stable-ready (с 1.11.0) | Canvas API / WebGL через Skia | `WebElementView()` для HTML-интеграции, навигация браузера |
| **Web (Kotlin/JS)** | Устаревший (Legacy) | Canvas API / DOM | Не рекомендуется для новых проектов — используйте Wasm |

### Важные нюансы платформ

**iOS (с 1.11.0):**
- Рендеринг через Skia поверх Metal/Core Animation — нативная производительность, поддержка ProMotion 120 Гц.
- Accessibility: VoiceOver, AssistiveTouch, Full Keyboard Access работают «из коробки» через Semantics API.
- Начиная с 1.11.0 — улучшенная интеграция с Xcode (улучшенный export framework, поддержка Swift packages через Kotlin 2.4).
- Известный нюанс: при очень сложных списках (1000+ элементов с динамической высотой) может наблюдаться небольшое отставание от SwiftUI. Решение — `LazyColumn` с явными `key` и `contentType` для переиспользования компонентов.

**Web (Wasm, с 1.11.0):**
- Использует Wasm GC (Garbage Collection) — требует современные браузеры (Chrome 119+, Firefox 120+, Safari 17.4+).
- `WebElementView()` позволяет встраивать настоящие HTML-элементы (например, `<video>`, `<iframe>`) внутрь Compose-дерева.
- Tree-shaking в Wasm-билдах существенно уменьшает размер бандла по сравнению с Kotlin/JS.

**Desktop (JVM):**
- Требуется JDK 11+ (рекомендуется 17 или 21 LTS).
- Поддержка нативного оконного меню, drag-and-drop с ОС, системных tray icons.
- Можно распространять через `jpackage` (JDK 17+) — нативные установщики `.msi`, `.dmg`, `.deb`.

---

## 21.3. Структура проекта и организация модулей

Типичная структура проекта Compose Multiplatform следует архитектуре Kotlin Multiplatform:

```
my-app/
├── composeApp/                    # Общий модуль (KMP) с Compose UI
│   ├── src/
│   │   ├── commonMain/            # Общий код для всех платформ
│   │   │   └── kotlin/
│   │   │       └── com/example/app/
│   │   │           ├── ui/        # Composable-функции
│   │   │           ├── viewmodel/ # ViewModel + StateFlow
│   │   │           └── di/        # Dependency Injection
│   │   ├── androidMain/           # Android-специфичный код
│   │   ├── iosMain/               # iOS-специфичный код
│   │   ├── desktopMain/           # Desktop-специфичный код
│   │   └── wasmJsMain/            # Web (Wasm) код
│   └── build.gradle.kts
├── iosApp/                        # iOS-приложение (Xcode проект)
│   └── iosApp.xcodeproj
├── androidApp/                    # Android-приложение (точка входа)
├── desktopApp/                    # Desktop-приложение
├── webApp/                        # Web-приложение
└── gradle/
    └── libs.versions.toml         # Версионный каталог (Version Catalog)
```

> **Изменение 2026:** JetBrains стандартизировала имя модуля `composeApp` (раньше было `shared`). Это рекомендованная структура для всех новых проектов, создаваемых через Kotlin Multiplatform wizard (kmp.jetbrains.com).

### Version Catalog (libs.versions.toml) — актуальный пример

```toml
[versions]
kotlin = "2.4.0"
compose-multiplatform = "1.11.0"
androidx-compose = "1.11.2"
androidx-lifecycle = "2.9.0"
kotlinx-coroutines = "1.11.0"
ktor = "3.4.0"
sqldelight = "2.1.0"
koin = "4.0.0"
navigation-compose = "2.9.0"

[libraries]
compose-ui = { module = "androidx.compose.ui:ui", version.ref = "androidx-compose" }
compose-material3 = { module = "androidx.compose.material3:material3", version.ref = "androidx-compose" }
compose-navigation = { module = "androidx.navigation:navigation-compose", version.ref = "navigation-compose" }
kotlinx-coroutines-core = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "kotlinx-coroutines" }
ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor" }
sqldelight-runtime = { module = "app.cash.sqldelight:runtime", version.ref = "sqldelight" }
koin-core = { module = "io.insert-koin:koin-core", version.ref = "koin" }

[plugins]
kotlin-multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
compose-multiplatform = { id = "org.jetbrains.compose", version.ref = "compose-multiplatform" }
sqldelight = { id = "app.cash.sqldelight", version.ref = "sqldelight" }
```

### Многомодульная архитектура

JetBrains обновила стандартную структуру KMP-проекта в 2026 году для согласования с AGP 9.0. В многомодульных проектах рекомендуется выделять:

- `core` — базовые утилиты, типы, extension-функции;
- `data` — репозитории, источники данных (API, DB), DTO;
- `domain` — use cases, бизнес-сущности;
- `feature-*` — отдельные фичи с UI и ViewModel (например, `feature-auth`, `feature-profile`);
- `composeApp` — точка сборки UI, навигация, DI-граф.

---

## 21.4. Навигация и управление состоянием

Начиная с Compose Multiplatform 1.10.0 (апрель 2026), AndroidX Navigation Compose поддерживается на всех платформах (Android, iOS, Desktop, Wasm). Это значит, что для новых проектов необязательно использовать сторонние библиотеки навигации (Decompose, Voyager) — официальная навигация работает cross-platform.

### Типобезопасная навигация с Navigation 3

```kotlin
import androidx.navigation.NavController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import kotlinx.serialization.Serializable

@Serializable
data class ProfileScreen(val userId: String)

@Serializable
object HomeScreen

@Serializable
data class EditTaskScreen(val taskId: String)

@Composable
fun AppNavigation() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = HomeScreen) {
        composable<HomeScreen> {
            HomeScreen(
                onNavigateToProfile = { userId ->
                    navController.navigate(ProfileScreen(userId))
                },
                onEditTask = { taskId ->
                    navController.navigate(EditTaskScreen(taskId))
                }
            )
        }
        composable<ProfileScreen> { backStackEntry ->
            val profile = backStackEntry.toRoute<ProfileScreen>()
            ProfileScreen(profile.userId)
        }
        composable<EditTaskScreen> { backStackEntry ->
            val args = backStackEntry.toRoute<EditTaskScreen>()
            EditTaskScreen(args.taskId)
        }
    }
}
```

**Ключевые преимущества типобезопасной навигации:**
- Маршруты определяются как `@Serializable` классы или объекты — опечатки исключены на этапе компиляции.
- Параметры передаются с проверкой типов — `userId: String` не может стать `Int` по ошибке.
- Deep linking поддерживается из коробки (на Android — через intent filters, на iOS — через URL schemes).
- IDE обеспечивает автодополнение и рефакторинг маршрутов.

### Когда выбирать Decompose или Voyager

Хотя Navigation Compose теперь работает cross-platform, альтернативы остаются актуальны:

| Библиотека | Когда выбирать |
|------------|----------------|
| **Navigation Compose (AndroidX)** | Стандартный выбор для новых проектов с 1.10+; знакомо Android-разработчикам; лучшая интеграция с deep links |
| **Decompose** | Если нужен строгий lifecycle, child-aware state retention, preview-friendly архитектура без навигационного фреймворка |
| **Voyager** | Простой API, основанный на stack-модели; хорош для быстрого старта; меньше boilerplate |

### Управление состоянием: UDF + StateFlow

Общепринятый подход — **Unidirectional Data Flow (UDF)**: состояние поднимается в `ViewModel`, UI получает его через `StateFlow.collectAsStateWithLifecycle()` (Android) или `StateFlow.collectAsState()` (другие платформы), события передаются вниз через коллбэки.

```kotlin
class TasksViewModel(
    private val repository: TaskRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow(TasksUiState())
    val uiState: StateFlow<TasksUiState> = _uiState.asStateFlow()

    fun loadTasks() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            repository.getAllTasks()
                .onSuccess { tasks -> 
                    _uiState.update { it.copy(tasks = tasks, isLoading = false) }
                }
                .onFailure { e -> 
                    _uiState.update { it.copy(error = e.message, isLoading = false) }
                }
        }
    }

    fun addTask(title: String) { /* ... */ }
    fun toggleTask(id: String) { /* ... */ }
}

@Stable
data class TasksUiState(
    val tasks: List<Task> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)
```

> **Подробнее об управлении состоянием:** см. раздел [32. Управление состоянием: глубокое погружение](32-state-management-deep-dive.md) — там разобраны Snapshot-модель, `derivedStateOf`, `rememberSaveable` и антипаттерны.

---

## 21.5. Новые возможности Compose Multiplatform 1.10–1.11

### Unified `@Preview` (с 1.10.0)

Раньше для превью Composable-функций приходилось либо использовать Android Studio (только для Android-таргета), либо писать отдельные preview-обёртки. С CMP 1.10 единая аннотация `@Preview` работает на всех платформах:

```kotlin
import org.jetbrains.compose.ui.tooling.preview.Preview

@Composable
fun TaskRow(task: Task) {
    /* ... */
}

@Preview
@Composable
fun TaskRowPreview() {
    MaterialTheme {
        TaskRow(task = Task(id = "1", title = "Пример задачи", completed = false))
    }
}
```

Превью рендерятся в IDE (Fleet или Android Studio с плагином Compose Multiplatform) для любой целевой платформы. Это особенно полезно для iOS-разработки, где раньше превью требовало запуска симулятора.

### Shared element improvements (с 1.11.0)

`SharedTransitionScope` получил улучшения в 1.11.0:
- Лучшая производительность при множественных shared elements на одном экране.
- Новый animation tooling в Layout Inspector — визуализация shared element transitions.
- Поддержка shared bounds (а не только shared content) — элементы могут «морфировать» между разными размерами.

### Trackpad events (с 1.11.0)

Добавлены события trackpad (важно для Desktop и Web):
- `Modifier.pointerInput { detectTrackpadGestures(...) }` — распознавание swipe, pinch, rotate с trackpad.
- Поддержка `PointerEventType.Scroll` с высокой точностью (pixel-level) для Desktop.

### Coroutine execution in tests (с 1.11.0)

Раньше тесты Compose, запускающие корутины (например, через `LaunchedEffect`), требовали `runTest` или ручного управления `TestDispatcher`. С 1.11 в Compose UI Test появился `ComposeUiTest.mainClock` с автоматическим advance — корутины в `LaunchedEffect` выполняются синхронно с тестовым clock, что упрощает тестирование.

### WindowInsetsRulers (с 1.10.3)

`WindowInsetsRulers` — utility для позиционирования UI-элементов на основе window insets (status bar, navigation bar, notches). Работает на всех платформах, включая iOS (Dynamic Island) и Desktop (title bar).

---

## 21.6. Платформенная специфика: что нужно знать

| Задача | Android | iOS | Desktop | Web (Wasm) |
|--------|---------|-----|---------|------------|
| Запуск корутин | `viewModelScope` + `Dispatchers.Main` | `Dispatchers.Main` (через Dispatchers.Main) | `Dispatchers.Main` (Swing/AWT) | `Dispatchers.Main` (Wasm) |
| Файлы | `java.io.File` | `NSFileManager` | `java.io.File` | `js()` + Blob API |
| Сеть | `OkHttp`/`Ktor Android` | `Darwin` (Ktor `Darwin` engine) | `Java` (Ktor `Java` engine) | `Js` (Ktor `Js` engine) |
| БД | `SQLDelight AndroidDriver` | `SQLDelight NativeSqliteDriver` | `SQLDelight JdbcSqliteDriver` | `SQLDelight WebWorkerDriver` |
| Шифрование | `EncryptedSharedPreferences` | Keychain | `java.security.KeyStore` | Web Crypto API |

> **Подробнее:** раздел [11. Платформенная интеграция](11-platform-integration.md) и [33. expect/actual паттерны](33-expect-actual-patterns.md) — там разобраны паттерны для доступа к платформенным API.

---

⬅️ [← Глубокое погружение в Kotlin](20-deep-dive-kotlin.md) | [К содержанию](README.md) | [M3: глубокое погружение →](22-deep-dive-md3.md) ➡️
