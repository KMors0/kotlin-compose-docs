# Compose Multiplatform

> Compose Multiplatform — фреймворк от JetBrains для создания кроссплатформенных UI-компонентов на Kotlin. Позволяет писать декларативный UI, который можно использовать на Android, iOS, Web и других платформах.

---

## 2.1. Установка и настройка проекта

1. **Инструменты:** установите IntelliJ IDEA (рекомендуется), JDK 11+, Gradle.
2. **Создание проекта:** в IDEA выберите шаблон _Compose Multiplatform Application_.
3. **Структура проекта:**
   - `shared` — общий код (UI, бизнес-логика).
   - `androidApp`, `iosApp` — платформенные модули.

**Файл `build.gradle.kts` (общий модуль):**
```kotlin
kotlin {
    jvm()
    js(BOTH)
    ios()

    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation(compose.runtime)
                implementation(compose.foundation)
                implementation(compose.material3)
            }
        }
    }
}
```

Структура проекта Compose Multiplatform следует принципу разделения общего и платформенного кода. Общий модуль `shared` содержит бизнес-логику и UI-компоненты, которые компилируются для каждой целевой платформы. Платформенные модули (`androidApp`, `iosApp`, `webApp`) содержат точку входа и платформенно-зависимый код, например, навигацию между экранами или доступ к системным сервисам.

---

## 2.2. Декларативный UI

Компоненты Compose описываются как функции, которые возвращают дерево элементов.

**Пример экрана:**
```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "Hello $name!", fontSize = 24.sp)
}
```

**Ключевые принципы:**
- **Композируемые функции** (`@Composable`) — описывают UI.
- **Состояние** (`mutableStateOf()`) — управляет динамическими данными.
- **Layout** — `Column`, `Row`, `Box` для компоновки.

Декларативный подход означает, что вы описываете _что_ должно быть на экране, а не _как_ это отрисовать. Когда данные изменяются, Compose автоматически перерисовывает только те компоненты, которые зависят от изменённых данных. Это делает код более предсказуемым и удобным для отладки по сравнению с императивным подходом, где вам приходится вручную обновлять каждый элемент при изменении состояния.

---

## 2.3. Управление состоянием

Состояние в Compose — это изменяемые значения, которые перестраивают UI при изменении.

**Пример счётчика:**
```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}
```

- `remember` — сохраняет состояние между перекомпоновками.
- `mutableStateOf` — создаёт наблюдаемое значение.

Управление состоянием — один из ключевых аспектов Compose. Когда значение `mutableStateOf` изменяется, Compose автоматически инициирует перекомпоновку всех `@Composable`-функций, которые читают это значение. Это гарантирует, что UI всегда отражает актуальное состояние данных. Для более сложных сценариев используйте `StateFlow` и `ViewModel`.

**Продвинутый пример с ViewModel:**
```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)
    val count: StateFlow<Int> = _count.asStateFlow()

    fun increment() { _count.value++ }
}

@Composable
fun CounterScreen(viewModel: CounterViewModel) {
    val count by viewModel.count.collectAsState()
    Button(onClick = { viewModel.increment() }) {
        Text("Count: $count")
    }
}
```

---

## 2.4. Платформенная логика

Чтобы разделить платформенный и общий код, используйте:
- **`expect/actual` модули** — для платформенных зависимостей.
- **`platform-specific` функции** в общем коде с условной компиляцией.

**Пример (файл `Platform.kt` в общем модуле):**
```kotlin
expect fun getPlatformName(): String
```

**Реализация для Android (`androidApp` модуль):**
```kotlin
actual fun getPlatformName() = "Android"
```

**Реализация для iOS (`iosApp` модуль):**
```kotlin
actual fun getPlatformName() = "iOS"
```

Механизм `expect/actual` — мощный инструмент для написания кроссплатформенного кода. В общем модуле вы объявляете `expect`-декларацию (функцию, класс, свойство), а каждый платформенный модуль предоставляет свою `actual`-реализацию. Это позволяет абстрагировать платформенные различия за единой интерфейс, сохраняя при этом полную типобезопасность.

---

## 2.5. Сборка и запуск

- Для Android: используйте стандартный процесс сборки в IDEA.
- Для iOS: откройте `.xcworkspace` в Xcode.
- Для Web: Gradle-задачи (`compose.webBrowser`, `compose.webJs`).

Процесс сборки зависит от целевой платформы. Для Android используется стандартный инструмент Gradle, который генерирует APK или AAB. Для iOS Compose Multiplatform генерирует нативный фреймворк, который интегрируется в Xcode-проект. Для Web компилятор Kotlin/JS генерирует JavaScript-бандл, который можно развернуть на любом веб-сервере.

---

⬅️ [← Основы Kotlin](01-kotlin-basics.md) | [К содержанию](README.md) | [Material Design 3 →](03-material-design-3.md) ➡️
