# Анимации и Motion

> Анимации — один из самых сильных аспектов Compose. В отличие от императивных фреймворков, где анимации требуют ручного управления таймерами и интерполяторами, Compose предоставляет декларативный API: вы описываете целевое состояние, а Compose автоматически вычисляет промежуточные кадры. Material Design 3 Expressive (стабилизирован в `compose-material3` 1.4.x в 2026 году) добавляет системный подход к motion через **MotionScheme** — физически корректные переходы, container transform, shared axis и предустановленные motion-токены.

---

## 29.1. Обзор API анимаций

Compose предоставляет несколько уровней API для анимаций, от простых до продвинутых:

| API | Сложность | Для чего |
|-----|-----------|----------|
| `animate*AsState()` | Низкая | Анимация одного значения (цвет, размер, позиция) |
| `AnimatedVisibility` | Низкая | Появление/исчезновение элементов |
| `updateTransition()` | Средняя | Ор켿рированная анимация нескольких свойств |
| `AnimatedContent` | Средняя | Анимированная смена содержимого |
| `Animatable` | Высокая | Полный контроль над анимацией, жесты |
| `rememberInfiniteTransition()` | Низкая | Бесконечные циклические анимации |

Выбирайте самый простой API, который решает задачу. Не используйте `Animatable` для простого изменения цвета — `animateColorAsState` сделает это в одну строку.

---

## 29.2. animate*AsState — анимация значений

Семейство функций `animate*AsState` анимирует переход от текущего значения к новому при каждом изменении. Возвращает `State<T>`, который можно использовать в `@Composable`.

```kotlin
import androidx.compose.animation.animateColorAsState
import androidx.compose.animation.core.*
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp

@Composable
fun AnimatedCard(isSelected: Boolean) {
    // Анимация цвета фона
    val backgroundColor by animateColorAsState(
        targetValue = if (isSelected) MaterialTheme.colorScheme.primaryContainer
                       else MaterialTheme.colorScheme.surface,
        animationSpec = spring(dampingRatio = Spring.DampingRatioMediumBouncy),
        label = "card_bg"
    )

    // Анимация размера (Elevation через shadow)
    val elevation by animateDpAsState(
        targetValue = if (isSelected) 8.dp else 2.dp,
        animationSpec = tween(durationMillis = 200),
        label = "card_elevation"
    )

    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp),
        colors = CardDefaults.cardColors(containerColor = backgroundColor),
        elevation = CardDefaults.cardElevation(defaultElevation = elevation)
    ) {
        Text("Animated Card", modifier = Modifier.padding(16.dp))
    }
}
```

Параметр `animationSpec` определяет, **как** происходит анимация. Доступны:
- `spring()` — физическая пружина, ощущается естественно, не требует указания длительности.
- `tween()` — линейная интерполяция с указанием длительности и easing-кривой.
- `keyframes()` — покадровая анимация с ключевыми кадрами.
- `snap()` — мгновенное изменение без анимации.

> **Важно:** Всегда указывайте `label` — это имя, которое отображается в Layout Inspector и помогает отлаживать анимации.

---

## 29.3. AnimatedVisibility — появление и исчезновение

`AnimatedVisibility` автоматически анимирует появление и исчезновение дочерних элементов. Когда `visible` становится `false`, дочерние элементы удаляются из дерева компоновки после завершения анимации выхода.

```kotlin
import androidx.compose.animation.*
import androidx.compose.animation.core.*
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.unit.dp

@Composable
fun ExpandableDetails(isExpanded: Boolean, content: @Composable () -> Unit) {
    AnimatedVisibility(
        visible = isExpanded,
        enter = fadeIn(animationSpec = tween(300)) +
               expandVertically(animationSpec = spring(stiffness = Spring.StiffnessMediumLow)),
        exit = fadeOut(animationSpec = tween(200)) +
              shrinkVertically(animationSpec = tween(200))
    ) {
        Card(
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp, vertical = 4.dp)
        ) {
            content()
        }
    }
}
```

**Доступные переходы входа (enter):**

| Переход | Описание |
|---------|----------|
| `fadeIn()` / `fadeOut()` | Плавное изменение прозрачности |
| `slideInHorizontally()` / `slideOutHorizontally()` | Горизонтальный сдвиг |
| `slideInVertically()` / `slideOutVertically()` | Вертикальный сдвиг |
| `expandIn()` / `shrinkOut()` | Расширение/сжатие от точки |
| `expandHorizontally()` / `shrinkHorizontally()` | Горизонтальное расширение |
| `expandVertically()` / `shrinkVertically()` | Вертикальное расширение |
| `scaleIn()` / `scaleOut()` | Масштабирование |

Переходы комбинируются оператором `+` — они выполняются параллельно. Для последовательного выполнения используйте `fadeIn().then(slideIn())`.

---

## 29.4. updateTransition — оркестрация нескольких анимаций

`updateTransition` управляет несколькими анимированными значениями одновременно, синхронизируя их. Идеально для состояний, которые меняют несколько свойств одновременно.

```kotlin
enum class CardState { Collapsed, Expanded }

@Composable
fun TransitionCard() {
    var state by remember { mutableStateOf(CardState.Collapsed) }
    val transition = updateTransition(targetState = state, label = "card_transition")

    val padding by transition.animateDp(label = "padding") { cardState ->
        if (cardState == CardState.Expanded) 24.dp else 8.dp
    }

    val cornerRadius by transition.animateDp(
        transitionSpec = { tween(durationMillis = 300) },
        label = "corner"
    ) { cardState ->
        if (cardState == CardState.Expanded) 24.dp else 12.dp
    }

    val elevation by transition.animateDp(
        transitionSpec = { spring(dampingRatio = 0.4f, stiffness = 200f) },
        label = "elevation"
    ) { cardState ->
        if (cardState == CardState.Expanded) 12.dp else 4.dp
    }

    Card(
        onClick = { state = if (state == CardState.Collapsed) CardState.Expanded else CardState.Collapsed },
        modifier = Modifier.padding(padding),
        elevation = CardDefaults.cardElevation(defaultElevation = elevation),
        shape = MaterialTheme.shapes.large.copy(topStart = cornerRadius)
    ) {
        Text("Click to toggle", modifier = Modifier.padding(16.dp))
    }
}
```

---

## 29.5. AnimatedContent — анимированная смена содержимого

`AnimatedContent` анимирует переход между разными `@Composable`-содержимыми. В отличие от `AnimatedVisibility` (показывает/скрывает один и тот же элемент), `AnimatedContent` плавно переходит между разными лейаутами.

```kotlin
import androidx.compose.animation.*
import androidx.compose.animation.core.*

@Composable
fun AnimatedCounter(count: Int) {
    AnimatedContent(
        targetState = count,
        transitionSpec = {
            // Входящее содержимое slides in снизу, исходящее — выходит вверх
            slideInVertically { it } + fadeIn(animationSpec = tween(150)) togetherWith
            slideOutVertically { -it } + fadeOut(animationSpec = tween(150))
        },
        label = "counter"
    ) { targetCount ->
        Text(
            text = "$targetCount",
            style = MaterialTheme.typography.displayLarge
        )
    }
}
```

`AnimatedContent` особенно полезен для: переключения вкладок, смены формата контента (список → карта), анимации счётчиков, переходов между типами элементов.

---

## 29.6. M3 Expressive Motion и MotionScheme

Material 3 Expressive вводит **MotionScheme** — систему motion-theming, аналогичную `ColorScheme`, но для анимаций. MotionScheme определяет предустановленные `AnimationSpec` для различных категорий движения: spatial (движение в пространстве — позиция, размер) и effects (визуальные эффекты — цвет, прозрачность).

### Два пресета MotionScheme

| Пресет | Описание | Когда использовать |
|--------|----------|--------------------|
| `MotionScheme.expressive()` | Более живые, пружинистые анимации с заметными overshoot'ами | Рекомендуется Material для большинства приложений; особенно для hero moments — ключевых взаимодействий |
| `MotionScheme.standard()` | Более сдержанные, предсказуемые анимации | Корпоративные приложения, accessibility-focused UI, сценарии где важна предсказуемость |

### Использование MotionScheme в коде

```kotlin
import androidx.compose.material3.expressive.MaterialExpressiveTheme
import androidx.compose.material3.expressive.MotionScheme
import androidx.compose.material3.MaterialTheme

// 1. Применяем Expressive-тему на верхнем уровне
@Composable
fun AppTheme(content: @Composable () -> Unit) {
    MaterialExpressiveTheme(
        colorScheme = if (isSystemInDarkTheme()) DarkColorScheme else LightColorScheme,
        motionScheme = MotionScheme.expressive(),
        content = content
    )
}

// 2. Используем motionScheme в компонентах
@Composable
fun ExpressiveAnimatedCard(isSelected: Boolean) {
    val motionScheme = MaterialTheme.motionScheme

    val backgroundColor by animateColorAsState(
        targetValue = if (isSelected) MaterialTheme.colorScheme.primaryContainer
                       else MaterialTheme.colorScheme.surface,
        animationSpec = motionScheme.defaultEffectsSpec(),  // из темы
        label = "card_bg"
    )

    val padding by animateDpAsState(
        targetValue = if (isSelected) 24.dp else 12.dp,
        animationSpec = motionScheme.defaultSpatialSpec(),  // из темы
        label = "card_padding"
    )

    Card(
        modifier = Modifier.padding(padding),
        colors = CardDefaults.cardColors(containerColor = backgroundColor)
    ) {
        Text("Expressive Card")
    }
}
```

### Motion tokens (для справки)

Material 3 Expressive определяет motion tokens на уровне дизайн-системы:
- `md.sys.motion.spring.fast.spatial` — быстрое пространственное движение
- `md.sys.motion.spring.default.spatial` — стандартное пространственное движение
- `md.sys.motion.spring.fast.effects` — быстрые эффекты (color, opacity)
- `md.sys.motion.spring.default.effects` — стандартные эффекты

В Compose API эти токены доступны через методы `MotionScheme` (`fastSpatialSpec()`, `defaultSpatialSpec()`, `fastEffectsSpec()`, `defaultEffectsSpec()`), ручная настройка spring-параметров обычно не требуется.

### Container Transform (превращение контейнера)
```kotlin
// M3-style container transform между списком и деталями
@OptIn(ExperimentalSharedTransitionApi::class)
@Composable
fun SharedTransitionScope.TaskListScreen(
    onTaskClick: (Task) -> Unit,
    selectedTask: Task?
) {
    val animatedVisibilityScope = this

    LazyColumn {
        items(tasks) { task ->
            TaskRow(
                task = task,
                onClick = { onTaskClick(task) },
                modifier = Modifier.sharedElement(
                    rememberSharedContentState(key = "task-${task.id}"),
                    animatedVisibilityScope
                )
            )
        }
    }
}
```

**Shared Axis (общая ось перехода):**
```kotlin
// Используется при навигации между экранами одного уровня
val sharedAxisEnter = slideInHorizontally(
    initialOffsetX = { fullWidth -> -fullWidth / 3 },
    animationSpec = spring(stiffness = 400, dampingRatio = 0.85f)
) + fadeIn(animationSpec = tween(200))

val sharedAxisExit = slideOutHorizontally(
    targetOffsetX = { fullWidth -> fullWidth / 3 },
    animationSpec = spring(stiffness = 400, dampingRatio = 0.85f)
) + fadeOut(animationSpec = tween(200))

NavHost(navController, startDestination = "list") {
    composable("list",
        enterTransition = { sharedAxisEnter },
        exitTransition = { sharedAxisExit }
    ) { TaskListScreen() }
}
```

---

## 29.7. Жестовые анимации (Animatable, draggable)

Для анимаций, управляемых жестами пользователя (свайп, перетаскивание), используйте `Animatable` — класс, который позволяет управлять анимационным значением напрямую.

```kotlin
import androidx.compose.animation.core.*
import androidx.compose.foundation.gestures.detectDragGestures
import androidx.compose.foundation.layout.offset
import androidx.compose.runtime.*
import androidx.compose.ui.input.pointer.pointerInput
import androidx.compose.ui.unit.IntOffset
import kotlin.math.roundToInt

@Composable
fun SwipeToDismissItem(
    onDismiss: () -> Unit,
    content: @Composable () -> Unit
) {
    val offsetX = remember { Animatable(0f) }

    Box(
        modifier = Modifier
            .offset { IntOffset(offsetX.value.roundToInt(), 0) }
            .pointerInput(Unit) {
                detectDragGestures(
                    onDragEnd = {
                        // Если свайнули достаточно далеко — dismiss, иначе — вернуть
                        if (offsetX.value > 300f) {
                            onDismiss()
                        } else {
                            scope.launch { offsetX.animateTo(0f, spring()) }
                        }
                    },
                    onHorizontalDrag = { change, dragAmount ->
                        change.consume()
                        scope.launch { offsetX.snapTo(offsetX.value + dragAmount) }
                    }
                )
            }
    ) {
        content()
    }
}
```

Ключевые методы `Animatable`: `snapTo()` — мгновенный переход (для следования за пальцем), `animateTo()` — анимированный переход к целевому значению, `stop()` — остановка текущей анимации.

> **Важно:** в примере выше `scope` должен быть `rememberCoroutineScope()`. В `pointerInput`-блоке `onDragEnd` и `onHorizontalDrag` не имеют suspend-контекста, поэтому для вызова `animateTo`/`snapTo` нужен `scope.launch { ... }`. Правильный паттерн:
>
> ```kotlin
> val scope = rememberCoroutineScope()
> val offsetX = remember { Animatable(0f) }
>
> Box(
>     modifier = Modifier
>         .offset { IntOffset(offsetX.value.roundToInt(), 0) }
>         .pointerInput(Unit) {
>             detectDragGestures(
>                 onDragEnd = {
>                     scope.launch {
>                         if (offsetX.value > 300f) onDismiss()
>                         else offsetX.animateTo(0f, spring())
>                     }
>                 },
>                 onHorizontalDrag = { change, dragAmount ->
>                     change.consume()
>                     scope.launch { offsetX.snapTo(offsetX.value + dragAmount) }
>                 }
>             )
>         }
> ) { content() }
> ```

---

## 29.8. Бесконечные анимации и производительность

```kotlin
import androidx.compose.animation.core.*

@Composable
fun LoadingIndicator() {
    val infiniteTransition = rememberInfiniteTransition(label = "loading")

    // Пульсирующий размер
    val scale by infiniteTransition.animateFloat(
        initialValue = 0.9f,
        targetValue = 1.1f,
        animationSpec = infiniteRepeatable(
            animation = tween(600, easing = EaseInOutCubic),
            repeatMode = RepeatMode.Reverse
        ),
        label = "pulse"
    )

    // Вращение
    val rotation by infiniteTransition.animateFloat(
        initialValue = 0f,
        targetValue = 360f,
        animationSpec = infiniteRepeatable(
            animation = tween(1000, easing = LinearEasing),
            repeatMode = RepeatMode.Restart
        ),
        label = "rotation"
    )

    Box(
        modifier = Modifier
            .size(48.dp * scale)
            .graphicsLayer { rotationZ = rotation }
    ) {
        CircularProgressIndicator()
    }
}
```

**Производительность анимаций:** Compose автоматически запускает анимации на отдельном потоке (animation coroutine) и обновляет UI на каждом кадре. Чтобы анимации были плавными (60 fps / 120 fps): избегайте создания новых объектов внутри `animationSpec`, используйте `remember` для lambdas, помечайте данные как `@Stable`, и применяйте `graphicsLayer {}` для анимаций трансформации (смещение, вращение, масштаб) — они выполняются на GPU без перекомпоновки.

---

## 29.9. Spring-физика

Spring-анимации (пружинные) чувствуются более естественно, чем tween с линейным easing. Они основаны на физической модели пружины с двумя параметрами:

```kotlin
import androidx.compose.animation.core.Spring
import androidx.compose.animation.core.spring

// Пружинные настройки
val bouncy = spring<Float>(
    dampingRatio = Spring.DampingRatioBouncy, // 0.2 — сильно пружинистая
    stiffness = Spring.StiffnessMedium           // 400 N/m — средняя жёсткость
)

val smooth = spring<Float>(
    dampingRatio = Spring.DampingRatioNoBouncy, // 1.0 — без колебаний
    stiffness = Spring.StiffnessLow              // 200 N/m — мягкая
)

// Предустановленные комбинации
val defaultSpring = spring<Float>()  // dampingRatio=0.75, stiffness=300
```

| Параметр | Влияние | Рекомендуемые значения |
|----------|---------|----------------------|
| `dampingRatio` | Затухание колебаний | 0.2 (bouncy) → 0.75 (default) → 1.0 (no bouncy) |
| `stiffness` | Скорость пружины | 200 (low) → 300 (medium) → 400+ (high) |

Используйте `spring()` по умолчанию для большинства анимаций — он подстраивается под длительность целевого изменения и всегда останавливается плавно. `tween()` используйте только когда нужна фиксированная длительность (например, для синхронизации нескольких анимаций).

> **Best practice (2026):** начиная с M3 Expressive, предпочтительный путь — использовать `MotionScheme.expressive()` и доставать `AnimationSpec` из темы через `MaterialTheme.motionScheme.defaultSpatialSpec()` и т.п. Это обеспечивает консистентность движения во всём приложении и автоматически применяет актуальные Material-токены без ручной настройки spring-параметров.

---

⬅️ [← Accessibility](28-accessibility.md) | [К содержанию](README.md) | [Миграция на KMP →](30-migration-guide.md) ➡️
