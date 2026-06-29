# Расширенные возможности навигации

> Навигация — один из ключевых аспектов любого приложения с несколькими экранами. В этой части рассматриваются подходы к организации навигации в кроссплатформенном приложении, передача параметров и работа с глубокими ссылками.

---

## 8.1. Мультиплатформенная навигация

Используйте `androidx.navigation-compose` с условной компиляцией:

**Общий код:**
```kotlin
sealed class Screen {
    object Tasks : Screen()
    object Details : Screen()
}

val navController = rememberNavController()

NavHost(navController, startDestination = Screen.Tasks.route) {
    composable(Screen.Tasks.route) { TasksScreen() }
    composable(Screen.Details.route) { DetailsScreen() }
}
```

Навигация в Compose Multiplatform строится вокруг концепции `NavController` и `NavHost`. `NavController` управляет стеком экранов и предоставляет методы для переходов (`navigate()`, `popBackStack()`), а `NavHost` определяет соответствие между маршрутами и `@Composable`-экранами. `sealed class` для маршрутов обеспечивает типобезопасность — компилятор проверит, что все маршруты обработаны в `NavHost`.

---

## 8.2. Альтернативные библиотеки

- **Compose Multiplatform Navigation** — лёгкая альтернатива.
- **Custom Navigation** — ручное управление стеком экранов.

Если `androidx.navigation-compose` кажется избыточным, можно использовать более лёгкие альтернативы. Compose Multiplatform Navigation от JetBrains предоставляет минимальный API для навигации между экранами. Для простых приложений с двумя-тремя экранами достаточно ручного управления стеком через `mutableStateOf<Stack<Screen>>`.

**Пример кастомной навигации:**
```kotlin
class Navigator {
    private val _backStack = mutableStateListOf<Screen>(Screen.Tasks)
    val currentScreen: State<Screen> = derivedStateOf { _backStack.last() }

    fun navigate(screen: Screen) { _backStack.add(screen) }
    fun pop() { if (_backStack.size > 1) _backStack.removeLast() }
}
```

---

## 8.3. Передача параметров

**Пример передачи ID задачи:**
```kotlin
composable(
    route = "details/{taskId}",
    arguments = listOf(navArgument("taskId") { type = NavType.StringType })
) { backStackEntry ->
    val taskId = backStackEntry.arguments?.getString("taskId") ?: ""
    DetailsScreen(taskId)
}
```

Передача параметров через навигацию позволяет передавать данные между экранами без создания глобальных хранилищ. Параметры кодируются прямо в URL-маршруте, что обеспечивает их сохранение при пересоздании процесса (например, при нехватке памяти на Android). Для сложных объектов используйте сериализацию в JSON и передачу как строкового параметра.

---

## 8.4. Глубокие ссылки (Deep Links)

- **Android:** настройте `intent-filter` в `AndroidManifest.xml`.
- **iOS:** используйте `URL Scheme` в `Info.plist`.
- **Shared:** обрабатывайте URL в общем коде.

**Пример для Android:**
```xml
<activity ...>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" android:host="tasks" />
    </intent-filter>
</activity>
```

Глубокие ссылки позволяют пользователям переходить на определённый экран приложения по URL. Это необходимо для push-уведомлений, отправки ссылок на контент и интеграции с другими приложениями. На Android глубокие ссылки настраиваются через `intent-filter`, на iOS — через `URL Scheme` или Universal Links. В общем коде вы можете обработать входящий URL и выполнить навигацию к нужному экрану.

**Пример обработки deep link в навигации:**
```kotlin
NavHost(
    navController = navController,
    startDestination = Screen.Tasks.route,
    deepLinks = listOf(
        navDeepLink { uriPattern = "myapp://tasks/{taskId}" }
    )
) {
    composable(
        route = "details/{taskId}",
        deepLinks = listOf(navDeepLink { uriPattern = "myapp://tasks/{taskId}" })
    ) { backStackEntry ->
        val taskId = backStackEntry.arguments?.getString("taskId") ?: ""
        DetailsScreen(taskId)
    }
}
```

---

⬅️ [← Локализация](07-localization.md) | [К содержанию](README.md) | [Тестирование →](09-testing.md) ➡️
