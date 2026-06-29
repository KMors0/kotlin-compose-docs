# Локализация и интернационализация

> Локализация (l10n) и интернационализация (i18n) — важные аспекты приложений, ориентированных на международную аудиторию. В Compose Multiplatform рекомендуемый подход — использование **Compose Resources** (начиная с версии 1.7.0), который предоставляет единый типобезопасный API для строковых ресурсов на всех платформах.

---

## 7.1. Compose Resources — современный подход

Compose Resources — встроенная система управления ресурсами в Compose Multiplatform, которая заменяет платформенные механизмы (strings.xml на Android, .strings на iOS, JSON на Web) единым кроссплатформенным API. Ресурсы размещаются в `commonMain/resources/` и автоматически генерируют типобезопасные классы.

### Структура файлов

```
composeApp/src/commonMain/resources/
├── values/
│   └── strings.xml          # Язык по умолчанию (английский)
├── values-ru/
│   └── strings.xml          # Русский
├── values-es/
│   └── strings.xml          # Испанский
└── values-zh/
│   └── strings.xml          # Китайский
```

### Файлы локализации

**`values/strings.xml` (язык по умолчанию):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">Task Manager</string>
    <string name="add_task">Add Task</string>
    <string name="delete_task">Delete Task</string>
    <string name="task_count">You have %d tasks</string>
    <string name="greeting">Hello, %s!</string>
    <string name="no_tasks">No tasks yet. Tap + to add one.</string>
</resources>
```

**`values-ru/strings.xml` (русский):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">Менеджер задач</string>
    <string name="add_task">Добавить задачу</string>
    <string name="delete_task">Удалить задачу</string>
    <string name="task_count">У вас %d задач</string>
    <string name="greeting">Привет, %s!</string>
    <string name="no_tasks">Задач пока нет. Нажмите +, чтобы добавить.</string>
</resources>
```

> **Форматирование:** Compose Resources поддерживает `%d` (числа), `%s` (строки) и позиционные аргументы (`%1$s`, `%2$d`) прямо в XML, аналогично Android strings.xml. Плюрализация (один/два/пять) поддерживается через `<plurals>` — см. ниже.

---

## 7.2. Использование в Compose UI

После добавления файлов и пересборки проекта (или запуска `./gradlew generateCommonMainComposeResourceGenerator`), Compose Multiplatform сгенерирует типобезопасный класс `Res.string.*`.

```kotlin
import org.jetbrains.compose.resources.stringResource
import mypackage.composeapp.generated.resources.Res
import mypackage.composeapp.generated.strings

@Composable
fun TaskListScreen(taskCount: Int) {
    Column(modifier = Modifier.padding(16.dp)) {
        // Простая строка
        Text(
            text = stringResource(Res.string.app_name),
            style = MaterialTheme.typography.headlineMedium
        )
        
        // Строка с параметром
        Text(
            text = stringResource(Res.string.greeting, "Alice"),
            style = MaterialTheme.typography.bodyLarge
        )
        
        // Строка с числом (форматирование %d)
        Text(
            text = stringResource(Res.string.task_count, taskCount),
            style = MaterialTheme.typography.bodyMedium
        )
    }
}
```

**Использование вне `@Composable` (в ViewModel, Use Cases):**

Для использования строковых ресурсов вне Compose-контекста используйте `org.jetbrains.compose.resources.getString`:

```kotlin
import org.jetbrains.compose.resources.getString

// В корутине или функции вне @Composable
suspend fun getNotificationTitle(): String {
    return getString(Res.string.new_task_notification)
}
```

---

## 7.3. Плюрализация (формы множественного числа)

Разные языки имеют разное количество форм множественного числа: русский — 3 формы (1 задача, 2 задачи, 5 задач), английский — 2 формы (1 task, 2 tasks), арабский — 6 форм. Compose Resources корректно обрабатывает это через элемент `<plurals>`.

```xml
<plurals name="tasks_remaining">
    <item quantity="one">Осталась %d задача</item>
    <item quantity="few">Осталось %d задачи</item>
    <item quantity="many">Осталось %d задач</item>
    <item quantity="other">Осталось %d задач</item>
</plurals>
```

```kotlin
@Composable
fun RemainingTasksText(count: Int) {
    Text(stringResource(Res.plurals.tasks_remaining, count, count))
    // count=1 → "Осталась 1 задача"
    // count=2 → "Осталось 2 задачи"
    // count=5 → "Осталось 5 задач"
}
```

**Английская версия (`values/strings.xml`):**
```xml
<plurals name="tasks_remaining">
    <item quantity="one">%d task remaining</item>
    <item quantity="other">%d tasks remaining</item>
</plurals>
```

---

## 7.4. Динамическая смена языка

Compose Resources определяет язык на основе системной локали устройства. Для ручного переключения языка внутри приложения используйте `CompositionLocal` и `LocalLocale`:

```kotlin
import org.jetbrains.compose.resources.ExperimentalResourceApi

// Создаём CompositionLocal для выбранного языка
val LocalAppLocale = staticCompositionLocalOf { Locale.current }

@OptIn(ExperimentalResourceApi::class)
@Composable
fun App() {
    var appLocale by remember { mutableStateOf(Locale.current) }
    
    CompositionLocalProvider(LocalAppLocale provides appLocale) {
        // UI с автоматически обновлёнными строками
        MainScreen(
            onLanguageChange = { newLocale -> appLocale = newLocale }
        )
    }
}
```

> **Примечание:** Для полной поддержки динамической смены языка на всех платформах может потребоваться использование библиотеки [lyricist](https://github.com/adrielcafe/lyricist) от Adriel Cafe, которая расширяет Compose Resources с более удобным API для переключения языков в рантайме, включая предварительную загрузку переводов.

---

## 7.5. Альтернативный подход: Lyricist

[Lyricist](https://github.com/adrielcafe/lyricist) — библиотека, упрощающая локализацию в Compose Multiplatform. Она генерирует типобезопасные строки из JSON-файлов и поддерживает переключение языков в рантайме без перезапуска приложения.

**Зависимость:**
```kotlin
implementation("cafe.adriel.lyricist:lyricist:1.7.0")
```

**JSON-файлы локализации (`strings/ru.json`):**
```json
{
  "app_name": "Менеджер задач",
  "greeting": "Привет, {name}!",
  "task_count": "У вас {count} задач"
}
```

```kotlin
@Composable
fun App() {
    val lyricist = rememberLyricist()
    
    // Переключение языка
    LaunchedEffect(Unit) {
        lyricist.language = LyricistLanguage.RU
    }
    
    // Использование
    Text(lyricist.strings.app_name)
    Text(lyricist.strings.greeting(name = "Alice"))
    Text(lyricist.strings.task_count(count = 5))
}
```

---

## 7.6. Форматирование дат и чисел

Используйте стандартные Kotlin-апи, которые работают кроссплатформенно:

```kotlin
import kotlinx.datetime.LocalDate
import kotlinx.datetime.format
import kotlinx.datetime.format.FormatStrings
import kotlinx.datetime.format.byUnicodePattern

@Composable
fun FormattedDate(date: LocalDate) {
    // Форматирование даты с учётом локали
    val formatted = date.format(LocalDate.Format {
        byUnicodePattern("dd MMM yyyy")
    })
    Text(formatted) // "30 июн 2026"
}

// Числа с разделителями по локали
fun formatNumber(value: Double): String {
    return "%,.2f".format(Locale.current, value) // "1 234,56" для ru-RU
}
```

---

## 7.7. Сравнение подходов

| Подход | Типобезопасность | Плюрализация | Смена языка в рантайме | Рекомендация |
|--------|------------------|-------------|------------------------|--------------|
| Compose Resources | Да (сгенерированные классы) | Да (`<plurals>`) | Ограниченная | Основной выбор |
| Lyricist | Да (сгенерированные классы) | Да (через ICU) | Полная | Если нужна смена языка без перезапуска |
| Ручной JSON/properties | Нет (строковые ключи) | Ручная | Полная | Не рекомендуется для новых проектов |
| kotlin-i18n | Частично | Да | Да | Устаревший подход, Java-стиль |

---

⬅️ [← Сетевые запросы](06-networking-api.md) | [К содержанию](README.md) | [Навигация →](08-navigation.md) ➡️