# Примеры реальных приложений

> Лучший способ закрепить знания — практика. В этой части рассматриваются три примера реальных приложений, демонстрирующих различные аспекты кроссплатформенной разработки на Kotlin Multiplatform.

---

## 16.1. Приложение-погодник

1. Интеграция с OpenWeatherMap API.
2. Кэширование прогноза на 24 часа.
3. Локализация названий городов.
4. Анимация изменения погоды (солнце → дождь).

Приложение-погодник — отличный пример проекта, который задействует большинство ключевых технологий: сетевые запросы (Ktor), локальное хранение (DataStore/SQLDelight), локализация и анимации. OpenWeatherMap API предоставляет JSON-ответы с данными о текущей погоде и прогнозе, которые нужно распарсить через `kotlinx.serialization`.

**Архитектура приложения:**
```
shared/
  ├── domain/
  │   ├── model/        # Weather, Forecast, Location
  │   └── usecase/      # GetWeatherUseCase, GetForecastUseCase
  ├── data/
  │   ├── remote/       # OpenWeatherMapApi
  │   ├── local/        # WeatherCache
  │   └── repository/   # WeatherRepositoryImpl
  └── presentation/
      ├── viewmodel/    # WeatherViewModel
      └── ui/           # WeatherScreen, ForecastScreen
```

**Ключевые реализации:**
- Кэширование прогноза на 24 часа через SQLDelight с автоматической инвалидацией по истечении срока.
- Анимация иконок погоды с помощью `AnimatedContent` — плавный переход между состояниями (солнце, облака, дождь).
- Локализация названий городов через API геокодинга.

---

## 16.2. Социальная сеть-мини

1. Авторизация через OAuth2 (Google, VK).
2. Лента постов с пагинацией (`LazyColumn`).
3. Комментарии с вложенными ответами.
4. Уведомления через Firebase (Android) / APNs (iOS).

Социальная сеть — более сложный пример, который требует работы с аутентификацией, пагинацией, вложенными структурами данных и push-уведомлениями. OAuth2 реализуется через Ktor-плагин `Auth` с поддержкой refresh-токенов. Пагинация ленты использует `LazyColumn` с бесконечной прокруткой — при достижении конца списка подгружается следующая страница.

**Пример пагинации:**
```kotlin
@Composable
fun FeedScreen(viewModel: FeedViewModel) {
    val posts by viewModel.posts.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()

    LazyColumn {
        items(posts, key = { it.id }) { post ->
            PostItem(post, onLike = { viewModel.likePost(post.id) })
        }
        if (isLoading) {
            item { CircularProgressIndicator() }
        }
    }

    // Бесконечная прокрутка
    LaunchedEffect(posts.size) {
        viewModel.loadMoreIfNeeded()
    }
}
```

Вложенные комментарии реализуются через рекурсивные `@Composable`-функции, а push-уведомления — через платформенные API с `expect/actual` для абстракции.

---

## 16.3. Калькулятор

1. Модель состояния для ввода чисел и операций.
2. Поддержка сложных выражений (BODMAS).
3. История вычислений с сохранением в локальном хранилище.
4. Тёмная/светлая тема с MD3.

Калькулятор — отличный пример приложения, где MVI-архитектура проявляет себя наилучшим образом. Каждое нажатие кнопки генерирует Intent, Reducer обрабатывает его и создаёт новое State, а UI просто отображает текущее State. Это делает логику вычислений предсказуемой и легко тестируемой.

**Пример модели состояния калькулятора:**
```kotlin
data class CalculatorState(
    val expression: String = "",
    val result: String = "0",
    val history: List<String> = emptyList()
)

sealed class CalculatorIntent {
    data class Digit(val digit: String) : CalculatorIntent()
    data class Operation(val op: String) : CalculatorIntent()
    object Equals : CalculatorIntent()
    object Clear : CalculatorIntent()
    object Backspace : CalculatorIntent()
}
```

История вычислений сохраняется в SQLDelight и синхронизируется между экранами через `StateFlow`. Тёмная и светлая тема переключаются через `MaterialTheme` с `isSystemInDarkTheme()` по умолчанию и возможностью ручного выбора.

---

⬅️ [← Отладка](15-debugging.md) | [К содержанию](README.md) | [Обновления экосистемы →](17-ecosystem-updates.md) ➡️
