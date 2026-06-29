# Производительность и оптимизация

> Производительность — критически важный аспект пользовательского опыта. Этот раздел описывает методы анализа и оптимизации Compose Multiplatform-приложений на 2026 год: минимизация перекомпоновок, кэширование, использование `contentType` в Lazy-списках, профилирование через Layout Inspector и актуальные возможности рендеринга через Skia M144.

---

## 10.1. Анализ перекомпоновок

Используйте `RecompositionCounted` и `Layout Inspector`:

```kotlin
@OptIn(ExperimentalComposeApi::class)
@Composable
fun CountedGreeting() {
    RecompositionCounted("Greeting") {
        Greeting("World")
    }
}
```

Перекомпоновка (recomposition) — это процесс повторного вызова `@Composable`-функций при изменении состояния. Хотя Compose оптимизирован для минимизации перекомпоновок, неэффективный код может привести к избыточным перерисовкам. Например, если `@Composable`-функция читает состояние, которое изменяется часто (например, позиция прокрутки), она будет перекомпоновываться на каждом кадре.

**Советы по снижению перекомпоновок:**
- Разделяйте стабильные и нестабильные части UI на отдельные `@Composable`-функции.
- Используйте `derivedStateOf` для создания производных состояний, которые изменяются реже.
- Применяйте `remember` для кэширования вычислений, которые не нужно пересчитывать при каждой перекомпоновке.
- Помечайте классы данных аннотацией `@Stable`, если они неизменяемы.

---

## 10.2. Кэширование данных

- `remember` — кэширует значения между перекомпоновками.
- `rememberLazyListState` — сохраняет позицию прокрутки в `LazyColumn`.

**Пример:**
```kotlin
val state = rememberLazyListState()
LazyColumn(state = state) { ... }
```

Кэширование в Compose реализуется через механизм `remember`. Когда Compose перекомпонует функцию, `remember` возвращает ранее сохранённое значение вместо создания нового. Это особенно важно для объектов, создание которых дорого (например, аниматоры, форматеры, вычислительные результаты). Для сохранения состояния при изменении конфигурации (поворот экрана на Android) используйте `rememberSaveable`.

**Пример кэширования вычислений:**
```kotlin
@Composable
fun ExpensiveList(items: List<Item>) {
    val sortedItems = remember(items) {
        items.sortedBy { it.priority }
    }
    LazyColumn {
        items(sortedItems) { item -> ItemRow(item) }
    }
}
```

> [!INFO]
> **`remember(items) { ... }`** — кэширование результата лямбды с зависимостью от `items`. Первый аргумент `items` — это «ключи»: если `items` не изменился с прошлой рекомпозиции, Compose вернёт сохранённый результат без перевыполнения лямбды. Если `items` изменился — лямбда выполнится заново. Без `remember` сортировка происходила бы при каждой recomposition, что дорого для больших списков.
>
> **`it` в `items.sortedBy { it.priority }`** — каждый элемент списка `items`. `sortedBy` принимает лямбду, которая для каждого элемента возвращает ключ сортировки. `it.priority` — извлекаем поле `priority` для сравнения.
>
> **`items(sortedItems) { item -> ItemRow(item) }`** — функция из `LazyColumn`, описывает, как рендерить элементы. `sortedItems` — список данных. Лямбда `{ item -> ItemRow(item) }` вызывается для каждого видимого элемента, получает его как `item` (имя задано явно, не `it`). Внутри лямбды можно создавать любой Composable-контент.

---

## 10.3. Оптимизация списков

- Используйте `LazyColumn` вместо `Column` для больших списков.
- Укажите `key()` для уникальных элементов:
  ```kotlin
  items(tasks, key = { it.id }) { task -> TaskItem(task) }
  ```
- Укажите `contentType()` для переиспользования компонентов (Compose Multiplatform 1.7+):
  ```kotlin
  items(tasks, key = { it.id }, contentType = { it.type }) { task ->
      TaskItem(task)
  }
  ```
- Минимизируйте вычисления внутри `@Composable`-функций.

`LazyColumn` — ключевой инструмент для отображения больших списков в Compose. В отличие от `Column`, которая рендерит все элементы сразу, `LazyColumn` загружает только видимые элементы и несколько элементов за пределами видимой области (prefetching). Указание `key` позволяет Compose эффективно обновлять список при изменениях — перемещении, добавлении или удалении элементов — без полной перерисовки.

**`contentType` (важно для гетерогенных списков):** если список содержит элементы разных типов (например, задачи, заголовки секций, баннеры), укажите `contentType` — Compose будет переиспользовать «слоты» того же типа, что снижает количество создаваемых компонент и улучшает производительность при быстром скролле.

**Дополнительные оптимизации:**
```kotlin
LazyColumn {
    items(
        count = tasks.size,
        key = { index -> tasks[index].id }
    ) { index ->
        val task = tasks[index]
        TaskItem(
            task = task,
            modifier = Modifier.animateItem() // с Compose 1.7+ (замена animateItemPlacement)
        )
    }
}
```

---

## 10.4. Профилирование

- **Android:** Android Studio Profiler → CPU/Memory.
- **iOS:** Instruments → Time Profiler.

Профилирование помогает выявить узкие места в производительности. На Android используйте Android Studio Profiler для анализа использования CPU, памяти и сети. Для Compose доступен специальный инструмент Layout Inspector, который показывает дерево компоновки и количество перекомпоновок для каждого компонента. На iOS используйте Instruments — мощный набор инструментов для профилирования, включая Time Profiler (CPU) и Allocations (память).

**Рекомендации по профилированию:**
- Профилируйте на реальных устройствах, а не на эмуляторах.
- Измеряйте производительность в release-сборке (с оптимизациями).
- Обращайте внимание на frame rate — он должен быть стабильным 60fps или 120fps.
- Используйте `Benchmark` библиотеку для автоматизированного тестирования производительности.

---

## 10.5. Бенчмарки (kotlinx-benchmark)

Для количественного измерения производительности кода используйте [kotlinx-benchmark](https://github.com/Kotlin/kotlinx-benchmark) — официальную библиотеку бенчмарков, работающую на JVM и Kotlin/Native.

**Настройка:**
```kotlin
// build.gradle.kts
plugins {
    id("org.jetbrains.kotlinx.benchmark") version "0.4.12"
}

benchmark {
    targets { register("desktopBenchmark") }
    configurations {
        named("desktopBenchmark") {
            iterations = 10
            warmups = 5
            iterationTime = 500
        }
    }
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-benchmark-runtime:0.4.12")
}
```

**Пример бенчмарка:**
```kotlin
import kotlinx.benchmark.*

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(BenchmarkTimeUnit.MICROSECONDS)
@State(Scope.Benchmark)
class SortingBenchmark {
    @Param("100", "1000", "10000")
    var size: Int = 0
    private lateinit var items: List<Task>

    @Setup
    fun setup() {
        items = List(size) { Task("$it", "Task $it", false) }
    }

    @Benchmark
    fun sortByPriority() = items.sortedBy { it.priority }

    @Benchmark
    fun filterAndSort() = items.filter { it.completed }.sortedByDescending { it.createdAt }
}
```

**Запуск:** `./gradlew :composeApp:desktopBenchmark`

**Android Macrobenchmark** — для измерения startup time и frame time на устройствах:
```kotlin
@LargeTest
@RunWith(AndroidJUnit4::class)
class ComposeStartupBenchmark {
    @get:Rule
    val benchmarkRule = MacrobenchmarkRule()

    @Test
    fun startup() = benchmarkRule.measureRepeated(
        packageName = "com.example.app",
        metrics = listOf(StartupTimingMetric()),
        iterations = 5,
        startupMode = StartupMode.COLD
    ) {
        pressHome()
        startActivityAndWait()
    }
}
```

Рекомендуется измерять: startup time (холодный/тёплый старт), scroll performance (frame time в LazyColumn), время рендеринга сложных экранов. Сравнивайте результаты между релизами для отслеживания регрессий.

---

## 10.6. Рендеринг через Skia M144

Compose Multiplatform рендерит UI через [Skia](https://skia.org/) — графическую библиотеку, также используемую Google Chrome, Flutter и Adobe products. Связка Skia с Kotlin называется [skiko](https://github.com/JetBrains/skiko) (Skia for Kotlin).

**Актуальная версия (CMP 1.11.1, июнь 2026):** Skia M144. Версия M144 принесла:
- Улучшенную производительность рендеринга текста (особенно для CJK-языков и арабской вязи).
- Оптимизации для сложных путей (SVG-графика, векторные иконки).
- Лучшую работу на iOS через Metal backend.
- Уменьшенный размер бинарника для Wasm-таргета.

### Проверка версии Skia

```kotlin
// В runtime можно проверить версию skiko
import org.jetbrains.skiko.Skiko

fun logSkikoVersion() {
    println("Skiko version: ${Skiko.version}")
    println("Skia version: ${org.jetbrains.skia.skiaVersion}")
}
```

### Backend'ы рендеринга

| Платформа | Backend | Особенности |
|-----------|---------|-------------|
| **Android** | OpenGL ES / Vulkan | Автоматический выбор по API level |
| **iOS** | Metal (через Skia) | Нативная производительность, 120 Гц ProMotion |
| **Desktop (JVM)** | OpenGL | Через GLFW для оконного режима |
| **Web (Wasm)** | WebGL 2 / WebGPU | WebGPU экспериментальный, требует Chrome 113+ |

> **Совет:** если видите проблемы с рендерингом (мерцание, чёрные артефакты, тиринг), проверьте — это может быть regression в новой версии Skia. Можно временно откатить skiko версию:
> ```toml
> # gradle/libs.versions.toml
> [versions]
> skiko = "0.9.20"  # явно указать версию
> ```

---

## 10.7. K2 compiler и производительность сборки

С Kotlin 2.4 + K2 compiler:
- **Чистая сборка** — до 94% быстрее, чем на Kotlin 1.9 (по бенчмаркам JetBrains).
- **Инкрементальная сборка** — улучшена обработка изменений в многомодульных проектах.
- **KSP2** — до 2x быстрее annotation processing (важно для проектов с Room, Moshi, Koin annotations).

Чтобы убедиться, что K2 включён:
```kotlin
// gradle.properties
kotlin.compiler.useK2=true  # по умолчанию true с Kotlin 2.0
```

---

⬅️ [← Тестирование](09-testing.md) | [К содержанию](README.md) | [Платформенная интеграция →](11-platform-integration.md) ➡️
