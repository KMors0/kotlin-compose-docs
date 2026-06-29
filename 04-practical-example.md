# Практический пример: приложение «Список задач»

> Создадим простое приложение с использованием Kotlin, Compose Multiplatform и MD3. Этот пример демонстрирует полный цикл разработки кроссплатформенного приложения — от модели данных до запуска на разных платформах.

---

## 4.1. Структура проекта

- **Общий модуль (`shared`):**
  - `Task.kt` — модель задачи.
  - `TasksViewModel.kt` — ViewModel для управления списком.
  - `TasksScreen.kt` — UI-экран.
- **Платформенные модули:**
  - `androidApp` — запуск Activity.
  - `iosApp` — запуск iOS-приложения.

Такая структура позволяет максимально переиспользовать код между платформами. Общий модуль содержит весь UI и бизнес-логику, а платформенные модули — лишь точку входа и конфигурацию, специфичную для конкретной платформы. Это один из главных преимуществ Compose Multiplatform: вы пишете UI один раз, а он работает везде.

---

## 4.2. Модель данных

```kotlin
data class Task(val id: String, val title: String, val completed: Boolean)
```

Модель данных — простая `data class`, которая автоматически получает реализации `equals()`, `hashCode()`, `toString()` и `copy()`. Поле `id` обеспечивает уникальную идентификацию задачи, `title` хранит текстовое описание, а `completed` — состояние выполнения. Использование `data class` также позволяет легко сравнивать объекты для определения изменений в списке, что важно для оптимизации перекомпоновок в `LazyColumn`.

---

## 4.3. ViewModel

Используем `mutableStateListOf()` для управления списком задач.

```kotlin
class TasksViewModel() {
    val tasks = mutableStateListOf<Task>()

    fun addTask(title: String) {
        tasks.add(Task(id = UUID.randomUUID().toString(), title = title, completed = false))
    }

    fun toggleTask(task: Task) {
        val index = tasks.indexOf(task)
        if (index != -1) {
            tasks[index] = task.copy(completed = !task.completed)
        }
    }
}
```

`mutableStateListOf()` — это наблюдаемая коллекция, которая автоматически уведомляет Compose об изменениях. Когда вы добавляете, удаляете или обновляете элемент списка, все `@Composable`-функции, которые читают этот список, будут перекомпонованы. Это избавляет от необходимости вручную управлять подписками и обновлениями UI.

---

## 4.4. UI-экран

```kotlin
@Composable
fun TasksScreen(viewModel: TasksViewModel) {
    Scaffold(floatingActionButton = {
        FloatingActionButton(
            onClick = { /* показать диалог добавления */ },
            containerColor = MaterialTheme.colorScheme.primary
        ) {
            Icon(Icons.Filled.Add, contentDescription = "Add task")
        }
    }) { innerPadding ->
        LazyColumn(modifier = Modifier.padding(innerPadding)) {
            items(viewModel.tasks) { task ->
                TaskItem(task = task, onToggle = { viewModel.toggleTask(task) })
            }
        }
    }
}

@Composable
fun TaskItem(task: Task, onToggle: () -> Unit) {
    Card(modifier = Modifier.padding(8.dp)) {
        Row(verticalAlignment = CenterVertically) {
            Checkbox(checked = task.completed, onCheckedChange = { onToggle() })
            Text(task.title, style = MaterialTheme.typography.titleMedium)
        }
    }
}
```

Экран списка задач использует `Scaffold` для базовой компоновки с `FloatingActionButton` для добавления новых задач. `LazyColumn` обеспечивает эффективную отрисовку списка — только видимые элементы загружаются в память, что критично для производительности при большом количестве задач. Каждый элемент списка оборачивается в `Card` для визуального разделения.

---

## 4.5. Запуск на Android

В `MainActivity.kt`:
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyAppTheme {
                val viewModel = TasksViewModel()
                TasksScreen(viewModel = viewModel)
            }
        }
    }
}
```

Для запуска на Android необходимо создать `ComponentActivity` и вызвать `setContent`, передав корневую `@Composable`-функцию. Обязательно оберните содержимое в тему (`MyAppTheme`), чтобы все компоненты использовали единую цветовую схему и типографику.

---

## 4.6. Тестирование на iOS

Откройте `.xcworkspace` в Xcode, выберите схему и запустите на симуляторе.

Для iOS Compose Multiplatform генерирует нативный фреймворк, который интегрируется в Xcode-проект. Общий код (UI и бизнес-логика) компилируется в Kotlin/Native и работает на iOS так же, как и на Android. Вам нужно лишь настроить точку входа в Xcode — остальное берёт на себя Gradle-плагин Compose Multiplatform.

---

⬅️ [← Material Design 3](03-material-design-3.md) | [К содержанию](README.md) | [Работа с данными →](05-data-storage.md) ➡️
