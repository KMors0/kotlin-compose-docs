# Глубокое погружение в Compose Multiplatform

> Детальное рассмотрение архитектуры Compose Multiplatform, матрицы поддержки платформ, структуры проекта и навигации — с актуальными данными на 2026 год.

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
                    text = "Compose Multiplatform allows this to run on Android, iOS, Desktop, and Web.",
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

Compose Multiplatform обеспечивает **полный паритет API** с Jetpack Compose для Android. Все layout-контейнеры (`Row`, `Column`, `Box`, `LazyColumn`, `LazyRow`), анимации (`animateContentSize`, `AnimatedVisibility`, `updateTransition`), жесты (`detectTapGestures`, `scrollable`, `draggable`) и системные API работают идентично на всех платформах.

---

## 21.2. Матрица поддержки платформ

Compose Multiplatform на середину 2026 года поддерживает следующие платформы:

| Платформа | Статус | Бэкенд рендеринга | Ключевые особенности |
|-----------|--------|-------------------|---------------------|
| **Android** | Stable (паритет с Jetpack Compose) | Android Canvas | Полный доступ к Android API, Material 3 |
| **iOS** | Stable (с 1.8.0, май 2025) | Metal / Core Animation (через Skia) | VoiceOver, AssistiveTouch, Full Keyboard Access |
| **Desktop (JVM)** | Stable | Skia (OpenGL) | Поддержка JDK 11+, нативное оконирование |
| **Web (Kotlin/Wasm)** | Beta (с 1.9.0, сен 2025) | Canvas API / WebGL | Интеграция с навигацией браузера, `WebElementView()` |
| **Web (Kotlin/JS)** | Alpha | Canvas API / DOM | Альтернативный таргет для веба |

Важно отметить, что Compose Multiplatform для iOS рендерит UI через Skia (графическую библиотеку, также используемую Google Chrome, Flutter и Adobe products), что обеспечивает нативную производительность и поддержку 120 Гц дисплеев. Согласно докладу из KTH Royal Institute of Technology, производительность Compose Multiplatform на iOS сопоставима с нативным SwiftUI для большинства сценариев, хотя при очень сложных списках (1000+ элементов) наблюдается небольшое отставание.

---

## 21.3. Структура проекта и организация модулей

Типичная структура проекта Compose Multiplatform следует архитектуре Kotlin Multiplatform:

```
my-app/
├── shared/                    # Общий модуль (KMP)
│   ├── src/
│   │   ├── commonMain/        # Общий код для всех платформ
│   │   │   └── kotlin/        # Компоненты UI, ViewModel, бизнес-логика
│   │   ├── androidMain/       # Android-специфичный код
│   │   ├── iosMain/           # iOS-специфичный код
│   │   ├── desktopMain/       # Desktop-специфичный код
│   │   └── wasmJsMain/        # Web-специфичный код
│   └── build.gradle.kts
├── androidApp/                # Android-приложение (точка входа)
├── iosApp/                    # iOS-приложение (Xcode проект)
├── desktopApp/                # Desktop-приложение
├── webApp/                    # Web-приложение
└── gradle/
    └── libs.versions.toml     # Версионный каталог (Version Catalog)
```

JetBrains обновила стандартную структуру проекта KMP в мае 2026 года для обеспечения более чётких обязанностей модулей и согласования с AGP 9.0. В много модульных проектах рекомендуется выделять отдельные модули для: `core` (базовые утилиты), `data` (репозитории, API), `domain` (use cases), `feature-*` (отдельные фичи с UI и ViewModel).

---

## 21.4. Навигация и управление состоянием

Compose Multiplatform предоставляет типобезопасную навигацию через официальную библиотеку `navigation-compose`. Начиная с версии 2.8.0 Navigation Compose, поддерживаются API с проверкой типов на этапе компиляции:

```kotlin
@Serializable
data class ProfileScreen(val userId: String)

@Serializable
object HomeScreen

@Composable
fun AppNavigation() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = HomeScreen) {
        composable<HomeScreen> {
            HomeScreen(
                onNavigateToProfile = { userId ->
                    navController.navigate(ProfileScreen(userId))
                }
            )
        }
        composable<ProfileScreen> { backStackEntry ->
            val profile = backStackEntry.toRoute<ProfileScreen>()
            ProfileScreen(profile.userId)
        }
    }
}
```

Ключевые преимущества типобезопасной навигации: маршруты определяются как `@Serializable` классы или объекты, параметры передаются с проверкой типов, deep linking поддерживается из коробки, а IDE обеспечивает автодополнение и рефакторинг маршрутов.

Для управления состоянием в Compose-приложениях общепринятым является подход **Unidirectional Data Flow (UDF)**, где состояние поднимается вверх по дереву компонентов, а события передаются вниз через коллбэки. В комбинации с `ViewModel` и корутинами это создаёт предсказуемую и тестируемую архитектуру.

---

⬅️ [← Глубокое погружение в Kotlin](20-deep-dive-kotlin.md) | [К содержанию](README.md) | [M3: глубокое погружение →](22-deep-dive-md3.md) ➡️
