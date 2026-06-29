# Стиль кода и документация

> Единый стиль кода и качественная документация — признаки зрелого проекта. В этой части рассматриваются рекомендации по оформлению Kotlin-кода, генерации API-документации и соглашения по именованию.

---

## 18.1. Kotlin Code Style

Следуйте рекомендациям JetBrains:
- Отступы: 4 пробела.
- Имена: `camelCase` для переменных, `PascalCase` для классов.
- Избегайте длинных функций (максимум 10 строк).

Стиль кода Kotlin от JetBrains — де-факто стандарт в индустрии. IntelliJ IDEA автоматически форматирует код по этим правилам. Для обеспечения единообразия в команде используйте `.editorconfig`-файл в корне проекта и ktlint для автоматической проверки стиля на CI.

**Пример .editorconfig:**
```ini
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 4
insert_final_newline = true

[*.kt]
max_line_length = 120
```

**Пример интеграции ktlint в Gradle:**
```kotlin
plugins {
    id("org.jlleitschuh.ktlint") version "11.6.1"
}

ktlint {
    android.set(true)
    outputColorName.set("RED")
}
```

---

## 18.2. Комментарии и Dokka

Генерируйте API-документацию с Dokka:

```kotlin
/**
 * ViewModel для управления списком задач.
 * @param repository — источник данных.
 */
class TasksViewModel(private val repository: TasksRepository) {
    // ...
}
```

Настройте в `build.gradle.kts`:
```kotlin
tasks.dokkaHtml {
    outputDirectory = file("docs")
}
```

Dokka — официальный генератор документации для Kotlin, аналог Javadoc. Он поддерживает KDoc-синтаксис (похожий на Javadoc, но с поддержкой расширений Kotlin) и генерирует HTML-документацию из исходного кода. Dokka понимает все конструкции Kotlin: extension functions, data classes, sealed classes, coroutines и другие.

**Расширенные возможности Dokka:**
```kotlin
/**
 * Добавляет новую задачу в список.
 *
 * @param title Название задачи. Не может быть пустым.
 * @return ID созданной задачи.
 * @throws IllegalArgumentException если [title] пустой.
 *
 * @sample com.example.app.TasksViewModelTest.testAddTask
 */
fun addTask(title: String): String {
    require(title.isNotBlank()) { "Title cannot be blank" }
    val task = Task(UUID.randomUUID().toString(), title, false)
    tasks.add(task)
    return task.id
}
```

---

## 18.3. Соглашения по именованию

- Файлы: `TasksScreen.kt`, `TasksViewModel.kt`.
- Пакеты: `com.example.app.shared.ui.tasks`.
- Классы: `Task`, `TaskRepository`.

Согласованные соглашения по именованию делают код более читаемым и предсказуемым. В Kotlin Multiplatform рекомендуется следующая структура пакетов:

```
com.example.app/
  ├── shared/
  │   ├── domain/model/          # Task, User
  │   ├── domain/usecase/        # GetTasksUseCase
  │   ├── domain/repository/     # TaskRepository (interface)
  │   ├── data/remote/           # TaskApi
  │   ├── data/local/            # TaskDao
  │   ├── data/repository/       # TaskRepositoryImpl
  │   └── presentation/
  │       ├── viewmodel/         # TasksViewModel
  │       ├── ui/                # TasksScreen
  │       ├── navigation/        # NavGraph
  │       └── theme/             # AppTheme, ColorScheme
  ├── android/                   # Android-специфичный код
  └── ios/                       # iOS-специфичный код
```

**Правила именования:**
- `@Composable`-функции экранов: `*Screen` (например, `TasksScreen`).
- `@Composable`-функции компонентов: описательное имя (например, `TaskItem`, `LoadingIndicator`).
- ViewModel: `*ViewModel` (например, `TasksViewModel`).
- Use Cases: `*UseCase` (например, `GetTasksUseCase`).
- Репозитории: `*Repository` для интерфейса, `*RepositoryImpl` для реализации.
- Тестовые классы: `*Test` (например, `TasksViewModelTest`).

---

⬅️ [← Обновления экосистемы](17-ecosystem-updates.md) | [К содержанию](README.md) | [Ресурсы →](19-resources.md) ➡️
