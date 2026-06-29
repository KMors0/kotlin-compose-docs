# Архитектурные паттерны

> Правильная архитектура — фундамент поддерживаемого и масштабируемого приложения. В этой части рассматриваются основные архитектурные паттерны, применимые в кроссплатформенных проектах на Kotlin Multiplatform.

---

## 14.1. Clean Architecture

Разделите код на слои:
1. **Domain** — бизнес-логика, модели, Use Cases.
2. **Data** — репозитории, сетевые клиенты, локальные хранилища.
3. **Presentation** — ViewModel, UI.

**Пример Use Case:**
```kotlin
interface GetTasksUseCase {
    suspend fun execute(): List<Task>
}

class GetTasksUseCaseImpl(private val repository: TaskRepository) : GetTasksUseCase {
    override suspend fun execute() = repository.getAllTasks()
}
```

Clean Architecture, предложенная Робертом Мартином (Uncle Bob), основана на принципе зависимостей: внутренние слои (Domain) не знают о внешних (Data, Presentation). Это означает, что бизнес-логика полностью независима от инфраструктуры и UI, что делает её легко тестируемой и переносимой. В контексте Kotlin Multiplatform Domain-слой находится в общем модуле и не содержит платформенных зависимостей.

**Структура модулей:**
```
shared/
  ├── domain/       # Чистый Kotlin, без зависимостей
  │   ├── model/    # Task, User
  │   ├── usecase/  # GetTasksUseCase
  │   └── repository/ # TaskRepository (interface)
  ├── data/         # Реализация репозиториев
  │   ├── remote/   # Ktor API клиент
  │   ├── local/    # SQLDelight
  │   └── repository/ # TaskRepositoryImpl
  └── presentation/ # ViewModel + Compose UI
```

---

## 14.2. MVI (Model-View-Intent)

Альтернатива MVVM для сложных состояний:
- **Model** — текущее состояние.
- **View** — отображает состояние.
- **Intent** — действия пользователя.

**Пример:**
```kotlin
data class TasksState(val tasks: List<Task>, val loading: Boolean)

sealed class TasksIntent {
    object LoadTasks : TasksIntent()
    data class AddTask(val title: String) : TasksIntent()
}

class TasksReducer {
    fun reduce(state: TasksState, intent: TasksIntent): TasksState = when (intent) {
        is LoadTasks -> state.copy(loading = true)
        is AddTask -> state.copy(tasks = state.tasks + Task(UUID.randomUUID().toString(), intent.title, false))
    }
}
```

MVI — однонаправленный архитектурный паттерн, который особенно хорошо сочетается с Compose. Поток данных выглядит так: пользователь совершает действие → создаётся Intent → Reducer создаёт новое State → UI отражает новое State. Такой подход делает поток данных предсказуемым и упрощает отладку, так как каждое состояние приложения может быть воспроизведено.

**Полный пример MVI с ViewModel:**
```kotlin
class TasksViewModel : ViewModel() {
    private val _state = MutableStateFlow(TasksState(emptyList(), false))
    val state: StateFlow<TasksState> = _state.asStateFlow()

    fun processIntent(intent: TasksIntent) {
        _state.value = TasksReducer().reduce(_state.value, intent)
    }
}

@Composable
fun TasksScreen(viewModel: TasksViewModel) {
    val state by viewModel.state.collectAsState()
    // Отображение state.tasks, state.loading
}
```

---

## 14.3. Dependency Injection

- **Koin** — лёгкий DI-контейнер.
- **Dagger-Hilt** — для Android.
- **Кроссплатформенный DI** — ручное внедрение или кастомные модули.

**Пример Koin:**
```kotlin
startKoin {
    modules(listOf(appModule))
}

val appModule = module {
    single { TasksRepository() }
    viewModel { TasksViewModel(get()) }
}
```

Koin — прагматичный DI-фреймворк для Kotlin, который не использует генерацию кода и рефлексию (в runtime). Вместо этого он использует DSL Kotlin для объявления зависимостей, что делает его быстрым и простым в настройке. Koin работает на всех платформах Kotlin Multiplatform, что делает его идеальным выбором для кроссплатформенных проектов.

**Пример Koin в Compose:**
```kotlin
@Composable
fun App() {
    KoinApplication(application = {
        modules(appModule)
    }) {
        val viewModel: TasksViewModel = koinViewModel()
        TasksScreen(viewModel)
    }
}
```

Для крупных Android-проектов, где требуется максимальная производительность DI, рассмотрите Dagger-Hilt. Однако он не поддерживает Kotlin Multiplatform, поэтому для кроссплатформенного кода используйте Koin или ручное внедрение зависимостей через конструкторы.

---

⬅️ [← DevOps и CI/CD](13-devops-cicd.md) | [К содержанию](README.md) | [Отладка →](15-debugging.md) ➡️
