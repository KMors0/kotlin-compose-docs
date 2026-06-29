# Тестирование

> Тестирование — неотъемлемая часть разработки надёжных приложений. В этой части рассматриваются различные уровни тестирования: юнит-тесты, UI-тесты и энд-ту-энд тесты, а также инструменты для анализа покрытия кода.

---

## 9.1. Юнит-тесты

Используйте JUnit для проверки бизнес-логики.

**Пример теста для ViewModel:**
```kotlin
@Test
fun testAddTask() {
    val viewModel = TasksViewModel()
    viewModel.addTask("Buy milk")
    assertEquals(1, viewModel.tasks.size)
    assertEquals("Buy milk", viewModel.tasks[0].title)
}
```

Юнит-тесты проверяют отдельные единицы кода в изоляции. Для Kotlin Multiplatform используйте `kotlin-test` — мультиплатформенную библиотеку тестирования, которая работает на всех целевых платформах. Для JVM-целевых также доступен JUnit 5. При тестировании ViewModel изолируйте её от внешних зависимостей (репозитории, API) с помощью интерфейсов и мок-объектов.

**Пример с мокированием:**
```kotlin
class TasksViewModelTest {
    private val repository = mockk<TaskRepository>()
    private val viewModel = TasksViewModel(repository)

    @Test
    fun `load tasks from repository`() = runTest {
        every { repository.getAllTasks() } returns flowOf(listOf(Task("1", "Test", false)))
        viewModel.loadTasks()
        assertEquals(1, viewModel.tasks.size)
    }
}
```

---

## 9.2. UI-тесты с Compose Test Library

Проверяйте состояния, клики, анимации.

**Настройка:**
```kotlin
implementation("androidx.compose.ui:ui-test-junit4:$compose_version")
```

**Пример:**
```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun testTaskList() {
    composeTestRule.setContent { TasksScreen(TasksViewModel()) }
    composeTestRule
        .onNodeWithText("Task 1")
        .assertIsDisplayed()
        .performClick()
}
```

Compose Test Library предоставляет декларативный API для тестирования UI-компонентов. Вы можете находить элементы по тексту, тегам или семантическим свойствам, выполнять действия (клики, ввод текста) и проверять состояние (отображение, наличие, включённость). Семантические свойства (`contentDescription`, `testTag`) особенно важны для доступности и тестирования.

**Расширенный пример:**
```kotlin
@Test
fun testAddTaskFlow() {
    composeTestRule.setContent {
        MyAppTheme { TasksScreen(TasksViewModel()) }
    }

    // Нажимаем FAB
    composeTestRule.onNodeWithContentDescription("Add task").performClick()

    // Вводим название задачи
    composeTestRule.onNodeWithText("Введите название").performTextInput("New task")

    // Сохраняем
    composeTestRule.onNodeWithText("Сохранить").performClick()

    // Проверяем, что задача добавлена
    composeTestRule.onNodeWithText("New task").assertIsDisplayed()
}
```

---

## 9.3. Энд-ту-энд тесты

- **Android:** Espresso.
- **iOS:** XCTest.
- **Web:** Selenium.

**Пример Espresso:**
```kotlin
onView(withText("Add Task")).perform(click())
onView(withId(R.id.task_title)).perform(typeText("Buy milk"))
onView(withText("Save")).perform(click())
onView(withText("Buy milk")).check(matches(isDisplayed()))
```

E2E-тесты проверяют работу приложения целиком — от запуска до завершения сценария. Они медленнее юнит- и UI-тестов, но дают наибольшую уверенность в работоспособности приложения. Рекомендуется выделять критические пользовательские сценарии (логин, покупка, создание контента) и покрывать их E2E-тестами, а остальное проверять на уровне юнит- и интеграционных тестов.

---

## 9.4. Покрытие кода

Используйте Kover для анализа покрытия:

```kotlin
plugins {
    id("org.jetbrains.kotlin.plugin.kover") version "0.6.5"
}

kover {
    coverageEngine = "kover"
}
```

Kover — инструмент от JetBrains для измерения покрытия кода в Kotlin-проектах. Он работает с JVM- и мультиплатформенными проектами, интегрируется с Gradle и генерирует отчёты в HTML, XML и других форматах. Рекомендуемый минимум покрытия для бизнес-логики — 80%, но помните, что 100% покрытие не гарантирует отсутствие багов — важнее качество тестов, чем их количество.

---

⬅️ [← Навигация](08-navigation.md) | [К содержанию](README.md) | [Производительность →](10-performance.md) ➡️
