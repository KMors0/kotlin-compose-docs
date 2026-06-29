# Управление состоянием: глубокое погружение

> Состояние — самое сложное понятие в Compose. Неправильное управление состоянием приводит к лишним перекомпоновкам, багам, утечкам памяти и трудноотлаживаемым проблемам. Этот раздел даёт полное понимание модели состояния Compose: от низкоуровневых примитивов (Snapshot) до production-паттернов (UDF + StateFlow). Все API актуальны на Compose Multiplatform 1.11.0 и kotlinx-coroutines 1.11.0.

---

## 32.1. Модель состояния Compose: Snapshot

Compose использует систему **Snapshot** для отслеживания изменений состояния. Snapshot — это нечто большее, чем «наблюдаемый объект»: это изолированная транзакция, которая фиксирует все изменения и атомарно уведомляет подписчиков.

**Ключевые концепции Snapshot:**
- **SnapshotMutableState** — реализация `mutableStateOf()`, регистрирует изменения в текущем snapshot.
- **SnapshotMutationPolicy** — контролирует, когда изменения считаются значимыми (например, `structuralEqualityPolicy()` по умолчанию, `referentialEqualityPolicy()` для стабильных объектов, `neverEqualPolicy()` для принудительной рекомпоновки).
- **Snapshot.withMutableSnapshot {}** — явное создание мутабельного контекста. Обычно не нужен — Compose создаёт его автоматически при recomposition.

**Custom mutation policy:**
```kotlin
import androidx.compose.runtime.*

data class Task(val id: String, val title: String, val completed: Boolean)

// По умолчанию — structural equality. Изменение одного поля → recomposition.
val task = mutableStateOf(
    Task("1", "Buy milk", false),
    policy = SnapshotMutationPolicy.structuralEqualityPolicy()
)

// referential — только если заменили весь объект (task.value = Task(...))
val taskRef = mutableStateOf(
    Task("1", "Buy milk", false),
    policy = SnapshotMutationPolicy.referentialEqualityPolicy()
)
```

Выбор политики влияет на производительность. Если у вас большой неизменяемый объект, который часто «пересоздаётся» с теми же данными, используйте `structuralEqualityPolicy()` — Compose пропустит перекомпоновку при равенстве. Если объект гарантированно стабильный и вы хотите ручного контроля — `referentialEqualityPolicy()`.

---

## 32.2. remember vs rememberSaveable vs rememberUpdatedState

| Примитив | Сохраняется при recomposition | Сохраняется при config change | Сохраняется при process death | Для чего |
|----------|------------------------------|------------------------------|-----------------------------|----------|
| `remember { }` | Да | Нет | Нет | Кэширование вычислений, объекты
| `rememberSaveable { }` | Да | Да | Нет (если Saver не реализован) | Ввод пользователя, позиция скролла |
| `rememberUpdatedState()` | Да (всегда обновляет) | Нет | Нет | Коллбеки в лямбдах, переданных в дочерние |
| `rememberCoroutineScope()` | Да | Нет | Нет | Запуск корутин из @Composable |

**rememberUpdatedState — частая ошибка:**
```kotlin
// ❌ Плохо — лямбда «захватывает» устаревшее значение
@Composable
fun BadTimer(onTimeout: () -> Unit) {
    val currentOnTimeout by remember { mutableStateOf(onTimeout) }
    // onTimeout устареет, если родитель перекомпонует с новой лямбдой
}

// ✅ Правильно — always has the latest value
@Composable
fun GoodTimer(onTimeout: () -> Unit) {
    val currentOnTimeout = rememberUpdatedState(onTimeout)
    // currentOnTimeout.value всегда актуален
}
```

**Custom Saver для rememberSaveable:**
```kotlin
@Composable
fun DatePicker() {
    var selectedDate by rememberSaveable(stateSaver = LocalDateSaver) {
        mutableStateOf(LocalDate.now())
    }
    // selectedDate сохраняется при повороте экрана
}

private val LocalDateSaver = Saver<LocalDate, String>(
    save = { it.toString() },
    restore = { LocalDate.parse(it) }
)
```

---

## 32.3. derivedStateOf — предотвращение лишних рекомпоновок

`derivedStateOf` создаёт вычисляемое состояние, которое обновляется только при реальном изменении результата. Это критически важно при чтении состояния, которое изменяется часто (например, позиция скролла).

**Когда НУЖНО использовать derivedStateOf:**
```kotlin
@Composable
fun TaskListHeader(tasks: List<Task>) {
    // ✅ Правильно — пересчитывается только когда меняется результат фильтра
    val completedCount by remember {
        derivedStateOf { tasks.count { it.completed } }
    }
    Text("Completed: $completedCount")
}

// ✅ Правильно — для LazyListState (изменяется каждый кадр при скролле)
val listState = rememberLazyListState()
val isAtEnd by remember {
    derivedStateOf { listState.canScrollForward.not() }
}
```

> [!INFO]
> **`by`** (property delegation) — ключевое слово Kotlin для делегирования свойства. `val x by remember { ... }` означает «значение `x` делегируется объекту, который возвращает `remember`». Вместо прямого доступа вы получаете геттер/сеттер через делегат. Эквивалент без `by`: `val state = remember { ... }; val x = state.value` — но `by` позволяет писать `x` напрямую, без `.value`.
>
> **`derivedStateOf { ... }`** — функция Compose, создающая вычисляемое состояние, которое обновляется только когда результат лямбды изменяется. Лямбда читает другие состояния (`tasks`, `listState`), и Compose автоматически подписывается на них.
>
> **`it`** в `tasks.count { it.completed }` — каждый элемент списка `tasks`. `count { predicate }` вызывает лямбду для каждого элемента и считает, для скольких вернулось `true`. `it` — автоматическое имя единственного параметра.
>
> **`.not()`** — логическое отрицание. Эквивалент `!listState.canScrollForward`, но в форме метода — полезно для chain-вызовов: `a.b().not()` вместо `!(a.b())`.

**Когда НЕ НУЖНО:**
```kotlin
// ❌ Избыточно — tasks.count { ... } вызывается при каждой перекомпоновке
// и так, derivedStateOf не даёт преимущества, если зависимости не меняются часто
val count by remember { derivedStateOf { tasks.size } }
```

Правило: `derivedStateOf` нужен, когда вы читаете **часто изменяющееся** состояние (scroll position, animation value, text input) и вычисляете из него **редко изменяющийся** результат.

---

## 32.4. StateFlow, SharedFlow и collectAsStateWithLifecycle

Для состояния, которое нужно разделять между несколькими экранами или доступно из вне Compose, используйте `StateFlow` в ViewModel.

**StateFlow в ViewModel:**
```kotlin
class TasksViewModel(private val repository: TaskRepository) : ViewModel() {
    private val _uiState = MutableStateFlow(TasksUiState())
    val uiState: StateFlow<TasksUiState> = _uiState.asStateFlow()

    init { loadTasks() }

    private fun loadTasks() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            repository.getAllTasks()
                .onSuccess { tasks -> _uiState.update { it.copy(tasks = tasks, isLoading = false) } }
                .onFailure { e -> _uiState.update { it.copy(error = e.message, isLoading = false) } }
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

> [!INFO]
> **`_uiState.update { it.copy(isLoading = true) }`** — атомарное обновление состояния. `update` — метод `MutableStateFlow`, который читает текущее значение, применяет к нему лямбду и записывает результат. `it` — текущее значение `TasksUiState`. `it.copy(isLoading = true)` создаёт копию с изменённым полем `isLoading` (data class автоматически генерирует метод `copy`). Атомарность важна при конкурентных обновлениях — `update` использует CAS (compare-and-swap) под капотом.
>
> **`_uiState` (с подчёркиванием) vs `uiState`** — конвенция Kotlin: приватное `MutableStateFlow` с подчёркиванием, публичное `StateFlow` без. Внешний код может только читать `uiState`, изменять состояние может только ViewModel через `_uiState`.
>
> **`asStateFlow()`** — преобразует `MutableStateFlow` в read-only `StateFlow`. Внешний код не может вызвать `_uiState.value = ...` — только ViewModel имеет доступ к мутабельной версии.
>
> **`@Stable`** — аннотация Compose, помечает класс как стабильный (immortal, поля не меняются после создания). Compose использует это для оптимизации recomposition — если два `@Stable` объекта равны, Compose пропускает recomposition.
>
> **`viewModelScope.launch { ... }`** — запуск корутины в scope ViewModel. Когда ViewModel уничтожается, все корутины в `viewModelScope` автоматически отменяются — нет утечек.

**Почему StateFlow, а не LiveData:**
- StateFlow — часть kotlinx.coroutines, работает на всех платформах (LiveData — только Android).
- StateFlow имеет initialValue — нет необходимости в обёртках.
- StateFlow поддерживает операторы Flow (map, filter, combine).

**collectAsStateWithLifecycle vs collectAsState:**
```kotlin
// ✅ Предпочтительно для Android — приостанавливает сбор в фоне
val state by viewModel.uiState.collectAsStateWithLifecycle()

// ⚠️ Продолжает собирать, даже если приложение в фоне
val state by viewModel.uiState.collectAsState()

// Для KMP (не Android) — collectAsState, т.к. Lifecycle недоступен
// В Decompose — используйте subscribeAsState() или приём из Decompose Lifecycle
```

> **KMP-нюанс (2026):** `collectAsStateWithLifecycle` доступен только на Android через `androidx.lifecycle:lifecycle-runtime-compose`. Для iOS/Desktop/Wasm используйте обычный `collectAsState()` — Compose Multiplatform автоматически приостанавливает recomposition когда окно не активно (через `Recomposer.State`), поэтому отдельный lifecycle-хук не нужен.

---

## 32.5. Snapshot.StateFlow — низкоуровневый StateFlow

В Compose есть экспериментальный `androidx.compose.runtime.Snapshot.StateFlow` — это StateFlow, который **интегрирован с системой Snapshot Compose**. В отличие от kotlinx.coroutines.StateFlow, он не требует `collectAsState()` — его значение читается напрямую как Compose-состояние.

```kotlin
import androidx.compose.runtime.Snapshot

// Создаём Snapshot-aware StateFlow
val counter = Snapshot.mutableStateFlow(0)

@Composable
fun Counter() {
    // Читаем напрямую — без collectAsState
    val count = counter.value
    Text("Count: $count")
    Button(onClick = { counter.value++ }) { Text("+") }
}
```

> **Предупреждение:** API `Snapshot.StateFlow` остаётся экспериментальным (требует `@OptIn(ExperimentalComposeApi::class)`). Для прикладного кода предпочитайте стандартный `kotlinx.coroutines.StateFlow` + `collectAsStateWithLifecycle()` (Android) или `collectAsState()` (другие платформы). `Snapshot.StateFlow` полезен в библиотеках, фреймворках и сложных интеграциях (например, Decompose использует аналогичный подход).

---

## 32.6. Паттерны: State hoisting, UDF, ViewModel

**State Hoisting (восходящее состояние):** правило «состояние должно жить как можно выше, но как можно ниже». Это значит: поднимайте состояние к ближайшему общему предку, который знает, как его изменить.

```kotlin
// ✅ Безсостоятельный (stateless) компонент — переиспользуемый, тестируемый
@Composable
fun TaskRow(
    task: Task,
    onToggle: () -> Unit,
    onDelete: () -> Unit,
    modifier: Modifier = Modifier
) {
    Row(modifier = modifier) {
        Checkbox(checked = task.completed, onCheckedChange = { onToggle() })
        Text(task.title, modifier = Modifier.weight(1f))
        IconButton(onClick = onDelete) {
            Icon(Icons.Default.Delete, contentDescription = "Delete")
        }
    }
}

// Состояние живёт в родителе
@Composable
fun TaskList(viewModel: TasksViewModel) {
    val state by viewModel.uiState.collectAsStateWithLifecycle()
    LazyColumn {
        items(state.tasks, key = { it.id }) { task ->
            TaskRow(
                task = task,
                onToggle = { viewModel.toggleTask(task.id) },
                onDelete = { viewModel.deleteTask(task.id) }
            )
        }
    }
}
```

**Unidirectional Data Flow (UDF):**

Поток данных идёт вниз (состояние → UI), события идут вверх (UI → ViewModel). Это однонаправленный цикл: State → View → Intent → Reducer/Processor → State.

---

## 32.7. Частые ошибки и антипаттерны

**1. Чтение состояния в условии if/when (неустойчивое чтение):**
```kotlin
// ❌ Compose не может правильно отследить, какие ветки зависят от состояния
if (isLoading) {
    LoadingIndicator()
} else {
    Content()
}

// ✅ Явно выносите условия
val showLoading by remember { derivedStateOf { isLoading && tasks.isEmpty() } }
```

**2. Запуск побочных эффектов напрямую в @Composable:**
```kotlin
// ❌ Запуск корутины при каждой recomposition
@Composable
fun BadScreen(viewModel: TasksViewModel) {
    viewModel.loadTasks() // Вызывается при каждой recomposition!
}

// ✅ LaunchedEffect запускается один раз (при ключе = Unit)
@Composable
fun GoodScreen(viewModel: TasksViewModel) {
    LaunchedEffect(Unit) { viewModel.loadTasks() }
}
```

**3. Мутация состояния после асинхронной операции без гарантии актуальности:**
```kotlin
// ❌ State может измениться пока мы ждали результат
@Composable
fun BadSearch() {
    var query by remember { mutableStateOf("") }
    LaunchedEffect(query) {
        val results = api.search(query) // query может устареть
        // results.apply {} — но query уже другой!
    }
}

// ✅ Проверяем актуальность через Snapshot
@Composable
fun GoodSearch() {
    var query by remember { mutableStateOf("") }
    LaunchedEffect(query) {
        val currentQuery = query
        val results = api.search(currentQuery)
        if (query == currentQuery) { // Важно — проверяем!
            // Применяем результаты
        }
    }
}
```

---

⬅️ [← Доступность](28-accessibility.md) | [К содержанию](README.md) | [expect/actual паттерны →](33-expect-actual-patterns.md) ➡️
