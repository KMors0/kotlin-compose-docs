# Тестирование

> Тестирование — неотъемлемая часть разработки надёжных приложений. В Kotlin Multiplatform тесты пишутся один раз и запускаются на всех целевых платформах. В этой части рассматриваются юнит-тесты, UI-тесты, E2E-тесты и инструменты анализа покрытия кода.

---

## 9.1. Мультиплатформенные юнит-тесты (kotlin-test)

Для Kotlin Multiplatform используйте `kotlin-test` — официальную мультиплатформенную библиотеку тестирования, которая работает на JVM, iOS (Native), JS/Wasm и Desktop без изменений.

**Настройка (`build.gradle.kts`):**
```kotlin
kotlin {
    sourceSets {
        val commonTest by getting {
            dependencies {
                implementation(kotlin("test"))          // kotlin-test — базовые утверждения
                implementation(kotlin("test-annotations-common")) // @Test, @Ignore
            }
        }
        val androidUnitTest by getting {
            dependencies {
                implementation(kotlin("test-junit"))    // JUnit-адаптер для Android
            }
        }
        val iosTest by getting {
            // kotlin-test работает "из коробки" на iOS
        }
    }
}
```

**Пример мультиплатформенного теста:**
```kotlin
// shared/src/commonTest/kotlin/.../TasksViewModelTest.kt
class TasksViewModelTest {

    @Test
    fun `initial state is empty`() {
        val viewModel = TasksViewModel(FakeTaskRepository())
        assertEquals(0, viewModel.tasks.size)
        assertEquals(false, viewModel.isLoading)
    }

    @Test
    fun `add task increases list size`() = runTest {
        val viewModel = TasksViewModel(FakeTaskRepository())
        viewModel.addTask("Buy milk")
        assertEquals(1, viewModel.tasks.size)
        assertEquals("Buy milk", viewModel.tasks[0].title)
    }

    @Test
    fun `toggle task changes completed state`() = runTest {
        val viewModel = TasksViewModel(FakeTaskRepository())
        viewModel.addTask("Buy milk")
        viewModel.toggleTask(viewModel.tasks[0].id)
        assertEquals(true, viewModel.tasks[0].completed)
    }

    @Test
    fun `delete task removes from list`() = runTest {
        val viewModel = TasksViewModel(FakeTaskRepository())
        viewModel.addTask("Buy milk")
        val taskId = viewModel.tasks[0].id
        viewModel.deleteTask(taskId)
        assertEquals(0, viewModel.tasks.size)
    }

    @Test
    fun `loading state is correct during fetch`() = runTest {
        val repository = FakeTaskRepository(delayMs = 100)
        val viewModel = TasksViewModel(repository)
        
        viewModel.loadTasks()
        assertEquals(true, viewModel.isLoading) // Загрузка началась
        
        advanceUntilIdle() // Дожидаемся завершения корутин
        assertEquals(false, viewModel.isLoading) // Загрузка завершена
    }
}
```

> [!TIP]
> **`runTest { ... }`** — функция из `kotlinx-coroutines-test`, запускает тестовый блок корутин с виртуальным временем. Внутри `runTest` корутины выполняются синхронно и предсказуемо — можно контролировать время без реальных задержек.
>
> **`advanceUntilIdle()`** — функция внутри `runTest`, продвигает виртуальное время вперёд, пока все запущенные корутины не завершатся. Без этого вызова `viewModel.loadTasks()` может не успеть выполниться к моменту проверки `assertEquals(false, viewModel.isLoading)`.
>
> **`@Test`** — аннотация JUnit/kotlin-test, отмечает метод как тестовый. Тест-раннеры (JUnit, kotlin-test) находят все методы с `@Test` и запускают их.
>
> **Имена тестов в обратных кавычках** — `loading state is correct during fetch` — Kotlin позволяет использовать произвольные имена (с пробелами) в обратных кавычках. Это делает тест-отчёты более читаемыми: «loading state is correct during fetch PASSED» вместо «loadingStateIsCorrectDuringFetch PASSED».

// Fake-реализация для тестов — работает на всех платформах
class FakeTaskRepository(
    private val delayMs: Long = 0
) : TaskRepository {
    private val tasks = mutableListOf<Task>()
    
    override suspend fun getAllTasks(): List<Task> {
        delay(delayMs)
        return tasks.toList()
    }
    
    override suspend fun addTask(title: String): Task {
        delay(delayMs)
        val task = Task(id = UUID().toString(), title = title, completed = false)
        tasks.add(task)
        return task
    }
    
    override suspend fun toggleTask(id: String) {
        val index = tasks.indexOfFirst { it.id == id }
        if (index >= 0) tasks[index] = tasks[index].copy(completed = !tasks[index].completed)
    }
    
    override suspend fun deleteTask(id: String) {
        tasks.removeAll { it.id == id }
    }
}
```

> **Почему Fake, а не Mock?** В мультиплатформенных тестах библиотеки мокирования (MockK, Mockative) могут не поддерживать все таргеты. Fake-реализации (ручные заглушки) работают везде и проще в отладке. Для JVM-тестов можно использовать MockK.

**Запуск тестов:**
```bash
./gradlew :composeApp:allTests              # Все тесты всех платформ
./gradlew :composeApp:desktopTest           # Только JVM/Desktop тесты
./gradlew :composeApp:iosSimulatorArm64Test # iOS тесты (требует macOS)
./gradlew :composeApp:androidUnitTest       # Android юнит-тесты
```

---

## 9.2. UI-тесты с Compose Test Library

Проверяйте состояния, клики, анимации и навигацию. Compose UI Testing работает на JVM (Desktop/Android) и не требует эмулятора для базовых проверок.

**Настройка:**
```kotlin
// В sourceSet, где запускаются UI-тесты (например, androidUnitTest или desktopTest)
dependencies {
    @OptIn(org.jetbrains.compose.ExperimentalComposeLibrary::class)
    implementation(compose.uiTest)
}
```

**Пример базового UI-теста:**
```kotlin
class TasksScreenTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `empty state is shown when no tasks`() {
        composeTestRule.setContent {
            TasksScreen(viewModel = TasksViewModel(FakeTaskRepository()))
        }
        composeTestRule
            .onNodeWithText("No tasks yet")
            .assertIsDisplayed()
    }

    @Test
    fun `task appears after adding`() {
        val viewModel = TasksViewModel(FakeTaskRepository())
        composeTestRule.setContent {
            TasksScreen(viewModel = viewModel)
        }

        composeTestRule.onNodeWithContentDescription("Add task").performClick()
        composeTestRule.onNodeWithText("Enter task title").performTextInput("Buy milk")
        composeTestRule.onNodeWithText("Save").performClick()

        composeTestRule
            .onNodeWithText("Buy milk")
            .assertIsDisplayed()
    }

    @Test
    fun `tapping task toggles completion`() {
        val viewModel = TasksViewModel(FakeTaskRepository())
        viewModel.addTask("Buy milk")
        
        composeTestRule.setContent {
            TasksScreen(viewModel = viewModel)
        }

        composeTestRule.onNodeWithText("Buy milk").performClick()
        // Проверяем, что задача теперь перечёркнута (зависит от реализации)
    }
}
```

**Поиск элементов — лучшие практики:**
```kotlin
// По тексту — для видимых строк
onNodeWithText("Save")
// По contentDescription — для иконок и кнопок без текста
onNodeWithContentDescription("Add task")
// По testTag — для элементов без видимого текста
onNodeWithTag("task_list")
// Комбинированный поиск
onNode(hasText("Delete") and hasClickAction())

// Используйте testTag в UI:
@Composable
fun TaskItem(task: Task, onToggle: () -> Unit) {
    Row(modifier = Modifier.testTag("task_item_${task.id}")) {
        Checkbox(checked = task.completed, onCheckedChange = { onToggle() })
        Text(task.title)
    }
}
```

**Тестирование асинхронных операций:**
```kotlin
@Test
fun `loading indicator shown during fetch`() = runTest {
    val repository = FakeTaskRepository(delayMs = 500)
    val viewModel = TasksViewModel(repository)
    
    composeTestRule.setContent {
        TasksScreen(viewModel = viewModel)
    }
    
    viewModel.loadTasks()
    composeTestRule
        .onNodeWithTag("loading_indicator")
        .assertIsDisplayed()
    
    advanceUntilIdle()
    composeTestRule
        .onNodeWithTag("loading_indicator")
        .assertDoesNotExist()
}
```

---

## 9.3. Мультиплатформенное мокирование (Mockative)

[Mockative](https://github.com/nomisRev/mockative) — мультиплатформенная библиотека мокирования, работающая на JVM, Native, JS и Wasm.

**Зависимость:**
```kotlin
// commonTest
implementation("io.mockative:mockative:2.4.1")
```

**Пример:**
```kotlin
class TasksViewModelMockativeTest {
    private val repository = mock<TaskRepository>()
    private val viewModel = TasksViewModel(repository)

    @Test
    fun `load tasks calls repository`() = runTest {
        // Настройка мока
        whenever(repository.getAllTasks())
            .thenReturn(listOf(Task("1", "Test", false)))

        viewModel.loadTasks()

        // Проверка вызова
        verify(repository).getAllTasks()
        assertEquals(1, viewModel.tasks.size)
    }
}
```

---

## 9.4. Энд-ту-энд тесты

E2E-тесты проверяют работу приложения целиком — от запуска до завершения сценария. Они медленнее юнит- и UI-тестов, но дают наибольшую уверенность в работоспособности. Для кроссплатформенных приложений E2E-тесты по-прежнему платформенно-специфичны, но общие тестовые сценарии можно описать один раз.

| Платформа | Инструмент | Запуск |
|-----------|-----------|--------|
| Android | Espresso / Compose Testing | `./gradlew :androidApp:connectedAndroidTest` |
| iOS | XCTest (XCUITest) | Через Xcode |
| Web (Wasm) | Playwright / Selenium | Node.js-скрипты |
| Desktop | Compose UI Test (JVM) | `./gradlew :composeApp:desktopTest` |

**Пример Espresso для Android:**
```kotlin
@RunWith(AndroidJUnit4::class)
class TasksE2ETest {
    @get:Rule
    val composeTestRule = createAndroidComposeRule<MainActivity>()

    @Test
    fun fullTaskFlow() {
        composeTestRule.onNodeWithContentDescription("Add task").performClick()
        composeTestRule.onNodeWithText("Enter title").performTextInput("Buy milk")
        composeTestRule.onNodeWithText("Save").performClick()
        composeTestRule.onNodeWithText("Buy milk").assertIsDisplayed()
        composeTestRule.onNodeWithText("Buy milk").performClick()
        // Проверяем, что задача отмечена
    }
}
```

Рекомендуется выделять критические пользовательские сценарии (авторизация, оформление заказа, создание контента) и покрывать их E2E-тестами, а остальное проверять на уровне юнит- и UI-тестов. Соотношение тестов: 70% юнит, 20% UI, 10% E2E.

---

## 9.5. Покрытие кода (Kover)

[Kover](https://kotlin.github.io/kotlinx-kover/) — официальный инструмент от JetBrains для измерения покрытия кода в Kotlin Multiplatform проектах.

**Настройка:**
```kotlin
plugins {
    id("org.jetbrains.kotlinx.kover") version "0.9.1"
}

// Проверка минимального покрытия
kover {
    reports {
        total {
            xml { onCheck = true }
            html { onCheck = true }
        }
    }
    
    // Фильтры: измеряем только свой код, не зависимости
    filters {
        excludes {
            classes("*ComposableSingletons*", "*_Factory*")
            packages("androidx.*", "kotlinx.*")
        }
    }
}

// Задача для проверки покрытия в CI
tasks.koverVerify {
    rule {
        bound { minValue = 80 } // Минимум 80% покрытия
    }
}
```

**Генерация отчёта:**
```bash
./gradlew koverHtmlReport    # HTML-отчёт (откройте в браузере)
./gradlew koverXmlReport     # XML для CI-интеграции
./gradlew koverVerify        # Проверка порога (упадёт, если < 80%)
```

Kover работает с мультиплатформенными проектами и генерирует агрегированный отчёт покрытия по всем source sets. Рекомендуемый минимум покрытия для бизнес-логики (Domain-слой) — 80-90%, для Presentation-слоя — 60-70%, для Data-слоя — 70-80%. Помните, что 100% покрытие не гарантирует отсутствие багов — важнее качество тестов и покрытие критических пользовательских сценариев.

---

## 9.6. Сводная таблица инструментов тестирования

| Инструмент | Платформы | Для чего |
|-----------|-----------|----------|
| `kotlin-test` | Все (JVM, Native, JS, Wasm) | Юнит-тесты, утверждения |
| Compose UI Test | JVM (Android, Desktop) | Тестирование UI-компонентов |
| Mockative | Все | Мультиплатформенное мокирование |
| MockK | Только JVM | Продвинутое мокирование для Android |
| Kover | Все | Покрытие кода |
| Espresso | Android | E2E-тесты на устройстве |
| XCTest | iOS | E2E-тесты на устройстве |

---

⬅️ [← Навигация](08-navigation.md) | [К содержанию](README.md) | [Производительность →](10-performance.md) ➡️