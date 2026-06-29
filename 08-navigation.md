# Расширенные возможности навигации

> Навигация — ключевой аспект любого приложения с несколькими экранами. Этот раздел описывает три варианта: AndroidX Navigation Compose (теперь работает на всех платформах KMP с Compose Multiplatform 1.10+), Decompose для продвинутых случаев, и кастомную навигацию для простых приложений. Также разобраны передача параметров и deep links.

---

## 8.1. AndroidX Navigation Compose — стандартный путь

Начиная с Compose Multiplatform 1.10.0 (апрель 2026), AndroidX Navigation Compose поддерживается на всех платформах (Android, iOS, Desktop, Wasm). С Navigation 2.8.0+ type-safe API стал стабильным — маршруты определяются через `@Serializable` классы и объекты, опечатки исключены на этапе компиляции.

> **Когда выбирать:** стандартный путь для новых KMP-проектов. Знакомо Android-разработчикам, лучшая интеграция с deep links, минимум boilerplate.

### Зависимости

```toml
[versions]
navigation-compose = "2.9.0"

[libraries]
navigation-compose = { module = "org.jetbrains.androidx.navigation:navigation-compose", version.ref = "navigation-compose" }
```

> **Важно:** для KMP используйте артефакт `org.jetbrains.androidx.navigation:navigation-compose` (JetBrains-форк), не `androidx.navigation:navigation-compose` (только Android).

### Type-safe навигация

```kotlin
import androidx.navigation.NavController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.toRoute
import kotlinx.serialization.Serializable

// Маршруты — @Serializable классы и объекты
@Serializable
object HomeRoute

@Serializable
data class TaskDetailRoute(val taskId: String)

@Serializable
data class EditTaskRoute(val taskId: String, val isNew: Boolean = false)

@Serializable
object SettingsRoute

@Composable
fun AppNavigation() {
    val navController = rememberNavController()

    NavHost(
        navController = navController,
        startDestination = HomeRoute
    ) {
        composable<HomeRoute> {
            HomeScreen(
                onTaskClick = { taskId ->
                    navController.navigate(TaskDetailRoute(taskId))
                },
                onSettingsClick = {
                    navController.navigate(SettingsRoute)
                }
            )
        }

        composable<TaskDetailRoute> { backStackEntry ->
            val route: TaskDetailRoute = backStackEntry.toRoute()
            TaskDetailScreen(
                taskId = route.taskId,
                onEdit = { navController.navigate(EditTaskRoute(route.taskId)) },
                onBack = { navController.popBackStack() }
            )
        }

        composable<EditTaskRoute> { backStackEntry ->
            val route: EditTaskRoute = backStackEntry.toRoute()
            EditTaskScreen(
                taskId = route.taskId,
                isNew = route.isNew,
                onBack = { navController.popBackStack() }
            )
        }

        composable<SettingsRoute> {
            SettingsScreen(onBack = { navController.popBackStack() })
        }
    }
}
```

**Преимущества type-safe API:**
- **Никаких строк-маршрутов** — `route = "task_detail/{taskId}"` заменено на классы.
- **Проверка типов на этапе компиляции** — `taskId: String` не может стать `Int` по ошибке.
- **IDE-поддержка** — автодополнение маршрутов, рефакторинг (rename), find usages.
- **Deep linking** — автоматически работает с `@Serializable` типами.

### Передача сложных параметров

С type-safe навигацией можно передавать не только строки, но и структуры:

```kotlin
@Serializable
data class FilterOptions(
    val showCompleted: Boolean,
    val category: String,
    val tags: List<String>  // Сериализуется в строку запроса
)

@Serializable
data class FilteredTasksRoute(val filter: FilterOptions)

// Передача
navController.navigate(FilteredTasksRoute(filter))

// Получение
composable<FilteredTasksRoute> { entry ->
    val route: FilteredTasksRoute = entry.toRoute()
    // route.filter.showCompleted, route.filter.tags
}
```

> **Ограничение:** все поля `@Serializable`-маршрута должны быть примитивами или `List`/`Map` примитивов. Передача вложенных объектов не работает напрямую — сериализуйте их в JSON-строку и передавайте как `String`.

### Анимации переходов

```kotlin
NavHost(
    navController = navController,
    startDestination = HomeRoute,
    enterTransition = { fadeIn(tween(200)) + slideInHorizontally { it / 4 } },
    exitTransition = { fadeOut(tween(200)) + slideOutHorizontally { -it / 4 } },
    popEnterTransition = { fadeIn(tween(200)) + slideInHorizontally { -it / 4 } },
    popExitTransition = { fadeOut(tween(200)) + slideOutHorizontally { it / 4 } }
) {
    // маршруты
}
```

> [!INFO]
> **`it` в `slideInHorizontally { it / 4 }`** — ширина экрана в пикселях. Функция `slideInHorizontally` принимает лямбду, в которую передаётся полная ширина контейнера, и лямбда возвращает смещение (offset) в пикселях. `it / 4` означает «сдвинуть на четверть ширины экрана». `-it / 4` — сдвинуть в обратную сторону.
>
> **`+` в `fadeIn(...) + slideInHorizontally(...)`** — оператор объединения анимаций. В Compose enter/exit-анимации комбинируются через `+`, они выполняются параллельно. Для последовательного выполнения есть `then()` — `fadeIn().then(slideInHorizontally())`.
>
> **Лямбды `enterTransition = { ... }`** — `NavHost` вызывает эту лямбду каждый раз, когда происходит переход, чтобы получить анимацию. Внутри лямбды доступен `AnimatedContentTransitionScope` через `this`, но в простых случаях он не нужен.

### Bottom navigation с вложенными стеками

```kotlin
@Composable
fun MainScreen() {
    val navController = rememberNavController()
    val currentRoute = navController.currentBackStackEntryAsState().value?.destination?.route

    Scaffold(
        bottomBar = {
            NavigationBar {
                NavigationBarItem(
                    selected = currentRoute == HomeRoute::class.qualifiedName,
                    onClick = { navController.navigate(HomeRoute) { launchSingleTop = true } },
                    icon = { Icon(Icons.Default.Home, "Home") },
                    label = { Text("Home") }
                )
                NavigationBarItem(
                    selected = currentRoute == SettingsRoute::class.qualifiedName,
                    onClick = { navController.navigate(SettingsRoute) { launchSingleTop = true } },
                    icon = { Icon(Icons.Default.Settings, "Settings") },
                    label = { Text("Settings") }
                )
            }
        }
    ) { padding ->
        NavHost(
            navController = navController,
            startDestination = HomeRoute,
            modifier = Modifier.padding(padding)
        ) {
            composable<HomeRoute> { HomeScreen() }
            composable<SettingsRoute> { SettingsScreen() }
        }
    }
}
```

---

## 8.2. Decompose — для продакшн-KMP

[Decompose](https://github.com/arkivanov/Decompose) от Arkadii Ivanov — навигационная библиотека для случаев, когда нужна строгая архитектура с явным жизненным циклом компонентов и сохранением состояния. В отличие от Navigation Compose, Decompose не зависит от Android-фреймворков и обеспечивает:

- **Component-ориентированную архитектуру** — каждый экран = компонент с собственным lifecycle.
- **Автоматическое сохранение состояния** при уничтожении компонента (аналог `savedInstanceState`).
- **Ленивую инициализацию** — дочерние компоненты создаются только когда они отображаются.
- **Поддержку нескольких стеков** навигации одновременно (например, табы с вложенной навигацией).

### Зависимости

```toml
[versions]
decompose = "3.3.0"

[libraries]
decompose-core = { module = "com.arkivanov.decompose:decompose", version.ref = "decompose" }
decompose-compose = { module = "com.arkivanov.decompose:extensions-compose", version.ref = "decompose" }
```

### Пример: Stack-навигация

```kotlin
import com.arkivanov.decompose.ComponentContext
import com.arkivanov.decompose.router.stack.ChildStack
import com.arkivanov.decompose.router.stack.StackNavigation
import com.arkivanov.decompose.router.stack.childStack
import com.arkivanov.decompose.router.stack.pop
import com.arkivanov.decompose.router.stack.push
import com.arkivanov.decompose.value.Value

// Конфигурации экранов
sealed interface Config {
    data object TaskList : Config
    data class TaskDetail(val taskId: String) : Config
}

// Корневой компонент
class RootComponent(
    componentContext: ComponentContext,
    private val taskRepository: TaskRepository
) : ComponentContext by componentContext {

    private val navigation = StackNavigation<Config>()

    val stack: Value<ChildStack<Config, Child>> = childStack(
        source = navigation,
        initialConfiguration = Config.TaskList,
        handleBackButton = true,
        childFactory = ::createChild
    )

    private fun createChild(config: Config, context: ComponentContext): Child = when (config) {
        Config.TaskList -> Child.TaskList(
            TaskListComponent(context, taskRepository) { taskId ->
                navigation.push(Config.TaskDetail(taskId))
            }
        )
        is Config.TaskDetail -> Child.TaskDetail(
            TaskDetailComponent(context, config.taskId, taskRepository) {
                navigation.pop()
            }
        )
    }

    sealed interface Child {
        data class TaskList(val component: TaskListComponent) : Child
        data class TaskDetail(val component: TaskDetailComponent) : Child
    }
}

// UI-часть
@Composable
fun RootContent(component: RootComponent) {
    Children(
        stack = component.stack,
        animation = stackAnimation(slide())
    ) { child ->
        when (val instance = child.instance) {
            is RootComponent.Child.TaskList -> TaskListContent(instance.component)
            is RootComponent.Child.TaskDetail -> TaskDetailContent(instance.component)
        }
    }
}
```

### Когда выбирать Decompose

| Сценарий | Decompose лучше |
|----------|----------------|
| Нужна строгая бизнес-логика на уровне компонента, не UI | ✅ |
| Нужно сохранять состояние между сессиями | ✅ |
| Несколько одновременных стеков (табы с вложенной навигацией) | ✅ |
| Сложные lifecycle-требования (например, выполнить cleanup при скрытии экрана) | ✅ |
| Нужна простая стековая навигация | ❌ Navigation Compose проще |
| Нужны deep links из коробки | ❌ Navigation Compose лучше |

---

## 8.3. Voyager — простой stack-based подход

[Voyager](https://github.com/adrielcafe/voyager) — альтернативная навигационная библиотека с концепцией «Screen как единый модуль». Каждый экран — это класс, реализующий интерфейс `Screen`, со своей логикой и UI. Меньше boilerplate, чем Decompose, но меньше контроля над архитектурой.

```kotlin
import cafe.adriel.voyager.core.screen.Screen
import cafe.adriel.voyager.navigator.Navigator

class TaskListScreen : Screen {
    @Composable
    override fun Content() {
        TaskListContent(onTaskClick = { id -> Navigator.push(TaskDetailScreen(id)) })
    }
}

class TaskDetailScreen(private val taskId: String) : Screen {
    @Composable
    override fun Content() {
        TaskDetailContent(taskId)
    }
}

// Использование
@Composable
fun App() {
    Navigator(TaskListScreen())
}
```

> **Когда выбирать Voyager:** для небольших проектов, где нужна простая stack-навигация без Decompose-сложности. Хорош для прототипирования.

---

## 8.4. Кастомная навигация (для простых приложений)

Для приложений с 2-3 экранами можно обойтись без библиотеки — просто `mutableStateListOf<Screen>`:

```kotlin
sealed interface Screen {
    data object TaskList : Screen
    data class TaskDetail(val taskId: String) : Screen
    data object Settings : Screen
}

class SimpleNavigator {
    private val _backStack = mutableStateListOf<Screen>(Screen.TaskList)
    val currentScreen: State<Screen> = derivedStateOf { _backStack.last() }

    fun navigate(screen: Screen) {
        _backStack.add(screen)
    }

    fun pop() {
        if (_backStack.size > 1) _backStack.removeLast()
    }

    fun popToRoot() {
        while (_backStack.size > 1) _backStack.removeLast()
    }
}

@Composable
fun App() {
    val navigator = remember { SimpleNavigator() }
    val screen by navigator.currentScreen

    BackHandler(enabled = screen !is Screen.TaskList) {
        navigator.pop()
    }

    when (val current = screen) {
        Screen.TaskList -> TaskListScreen(
            onTaskClick = { navigator.navigate(Screen.TaskDetail(it)) },
            onSettingsClick = { navigator.navigate(Screen.Settings) }
        )
        is Screen.TaskDetail -> TaskDetailScreen(
            taskId = current.taskId,
            onBack = { navigator.pop() }
        )
        Screen.Settings -> SettingsScreen(
            onBack = { navigator.pop() }
        )
    }
}
```

> **Когда подходит:** прототипы, одноэкранные приложения с диалогами, приложения, где каждый экран — это просто `@Composable` функция без ViewModel.

---

## 8.5. Глубокие ссылки (Deep Links)

Глубокие ссылки позволяют переходить на конкретный экран приложения по URL. Это необходимо для push-уведомлений, отправки ссылок на контент и интеграции с другими приложениями.

### Настройка на платформах

**Android (`AndroidManifest.xml`):**
```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" android:host="tasks" />
    </intent-filter>
    <!-- App Links для https -->
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https" android:host="myapp.com" />
    </intent-filter>
</activity>
```

**iOS (`Info.plist`):**
```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>
```

Для Universal Links (https://) дополнительно настройте Associated Domains в Xcode: `applinks:myapp.com`.

### Обработка deep links в Navigation Compose

С type-safe навигацией deep links работают автоматически — нужно только объявить `navDeepLink`:

```kotlin
NavHost(navController, startDestination = HomeRoute) {
    composable<TaskDetailRoute>(
        deepLinks = listOf(
            navDeepLink<TodoRoute>(basePath = "myapp://tasks")
        )
    ) { entry ->
        val route = entry.toRoute<TaskDetailRoute>()
        TaskDetailScreen(route.taskId)
    }
}
```

Теперь URL `myapp://tasks/123` автоматически откроет `TaskDetailScreen(taskId = "123")`.

### Получение deep link извне (push notification)

```kotlin
// commonMain
expect class DeepLinkHandler {
    fun observeDeepLinks(): Flow<String>
}

// androidMain
actual class DeepLinkHandler(private val context: Context) {
    actual fun observeDeepLinks(): Flow<String> = callbackFlow {
        val component = ComponentActivity()  // или ваш host
        component.addOnNewIntentListener { intent ->
            intent.data?.toString()?.let { url -> trySend(url) }
        }
        awaitClose { }
    }
}

// Использование в навигации
@Composable
fun App() {
    val navController = rememberNavController()
    val deepLinkHandler = remember { DeepLinkHandler() }

    LaunchedEffect(Unit) {
        deepLinkHandler.observeDeepLinks().collect { url ->
            navController.handleDeepLink(url)
        }
    }

    NavHost(navController, startDestination = HomeRoute) {
        // маршруты
    }
}
```

---

## 8.6. Сводная таблица выбора навигационной библиотеки

| Библиотека | Сложность | Когда выбирать | Платформы | State retention |
|------------|-----------|----------------|-----------|-----------------|
| **Navigation Compose (AndroidX)** | Низкая | Новые KMP-проекты, простые приложения, важны deep links | Все | Limited (через rememberSaveable) |
| **Decompose** | Высокая | Production-KMP с сложной бизнес-логикой, строгий lifecycle | Все | Excellent (через ComponentContext) |
| **Voyager** | Средняя | Быстрый старт, средние проекты, без Decompose-сложности | Все | Хорошее |
| **Кастомная** | Низкая | Прототипы, очень простые приложения | Все | Зависит от реализации |

> **Best practice (2026):** для новых проектов — Navigation Compose (теперь KMP-compatible). Если на старте знаете, что нужна сложная бизнес-логика с lifecycle — Decompose. Voyager и кастомная навигация — для нишевых случаев.

---

⬅️ [← Локализация](07-localization.md) | [К содержанию](README.md) | [Тестирование →](09-testing.md) ➡️
