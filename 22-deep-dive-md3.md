# Material Design 3: глубокое погружение

> Детальное рассмотрение дизайн-токенов, компонентной библиотеки, M3 Expressive и адаптивного дизайна — с примерами кода для Compose Multiplatform 1.11.0 и compose-material3 1.4.x (актуально на 30 июня 2026).

---

## 22.1. Дизайн-токены и система тем

Material Design 3 основан на системе **дизайн-токенов** — семантически именованных значений, которые определяют внешний вид приложения. В Compose M3 эти токены представлены объектами `ColorScheme`, `Typography` и `Shape`:

```kotlin
import androidx.compose.material3.*

// Определение цветовой схемы
val LightColorScheme = lightColorScheme(
    primary = Color(0xFF6750A4),
    onPrimary = Color(0xFFFFFFFF),
    primaryContainer = Color(0xFFEADDFF),
    onPrimaryContainer = Color(0xFF21005D),
    secondary = Color(0xFF625B71),
    tertiary = Color(0xFF7D5260),
    error = Color(0xFFB3261E),
    background = Color(0xFFFFFBFE),
    surface = Color(0xFFFFFBFE),
    onBackground = Color(0xFF1C1B1F),
    onSurface = Color(0xFF1C1B1F),
)

val DarkColorScheme = darkColorScheme(
    primary = Color(0xFFD0BCFF),
    onPrimary = Color(0xFF381E72),
    primaryContainer = Color(0xFF4F378B),
    // ... остальные роли
)

// Применение темы
@Composable
fun AppTheme(content: @Composable () -> Unit) {
    MaterialTheme(
        colorScheme = if (isSystemInDarkTheme()) DarkColorScheme else LightColorScheme,
        typography = AppTypography,
        shapes = AppShapes,
        content = content
    )
}
```

Система M3 определяет **семантические цветовые роли**: `primary` (основной цвет), `secondary` (дополнительный), `tertiary` (третий акцент), `error` (ошибки), `surface` (поверхности), `background` (фон) — и их варианты `on*` (для текста/иконок поверх) и `*Container` (для контейнеров). Это позволяет адаптировать палитру под бренд, не меняя семантику цветов в коде.

### Динамический цвет (Dynamic Color)

На Android 12+ динамический цвет автоматически генерирует цветовую схему на основе обоев пользователя. В Compose Multiplatform динамический цвет работает только на Android; для других платформ используются статические схемы.

```kotlin
@Composable
fun AppTheme(content: @Composable () -> Unit) {
    val context = LocalContext.current
    val isAndroid12Plus = Build.VERSION.SDK_INT >= Build.VERSION_CODES.S

    val colorScheme = when {
        isAndroid12Plus -> {
            if (isSystemInDarkTheme()) dynamicDarkColorScheme(context)
            else dynamicLightColorScheme(context)
        }
        isSystemInDarkTheme() -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(colorScheme = colorScheme, content = content)
}
```

> **Best practice для KMP-проектов:** всегда имейте fallback на статические `LightColorScheme`/`DarkColorScheme` — динамические цвета доступны только на Android 12+.

---

## 22.2. Библиотека основных компонентов

Material Design 3 в Compose предоставляет обширный набор компонентов, организованных по категориям:

### Кнопки

| Компонент | Назначение |
|-----------|------------|
| `Button` | Основная кнопка с заливкой primary |
| `FilledTonalButton` | Вторичная кнопка с secondaryContainer |
| `OutlinedButton` | Третичная кнопка с обводкой |
| `TextButton` | Текстовая кнопка для низкоприоритетных действий |
| `ElevatedButton` | Кнопка с elevation для акцента |
| `IconButton` | Квадратная кнопка-иконка |
| `FilledIconButton` / `OutlinedIconButton` | Вариации IconButton |
| `FloatingActionButton` | FAB для главного действия экрана |
| `ExtendedFloatingActionButton` | FAB с текстом |

### Контейнеры

`Card`, `ElevatedCard`, `FilledCard`, `OutlinedCard`, `Surface`, `Scaffold`

### Навигация

`NavigationBar`, `NavigationRail`, `NavigationDrawer`, `NavigationItem`, `TabRow`, `ScrollableTabRow`

### Диалоги и окна

`AlertDialog`, `ModalBottomSheet`, `DatePicker`, `TimePicker`, `SegmentedButton`

### Ввод данных

`TextField`, `OutlinedTextField`, `FilledTextField`, `DropdownMenu`, `ExposedDropdownMenuBox`, `Slider`, `Switch`, `Checkbox`, `RadioButton`

### Отображение данных

`ListItem`, `HorizontalDivider` (ранее `Divider`), `ProgressIndicator`, `CircularProgressIndicator`, `LinearProgressIndicator`, `Badge`, `AssistChip`, `FilterChip`, `SuggestionChip`, `InputChip`

### Типографика

M3 определяет шкалу типографики с 15 ролями: `displayLarge`, `displayMedium`, `displaySmall`, `headlineLarge`, `headlineMedium`, `headlineSmall`, `titleLarge`, `titleMedium`, `titleSmall`, `bodyLarge`, `bodyMedium`, `bodySmall`, `labelLarge`, `labelMedium`, `labelSmall`.

Каждый компонент M3 в Compose следует принципам **elevation** через систему **tonal elevation** — вместо традиционных теней M3 использует смешивание цвета поверхности с цветом `primary`, создавая более мягкий и современный визуальный эффект.

> **Изменение в compose-material3 1.4:** `Divider` переименован в `HorizontalDivider` (старое имя deprecated). Также добавлен `VerticalDivider` для вертикальных разделителей.

---

## 22.3. M3 Expressive: новейшая эволюция

Material 3 Expressive — расширение Material Design 3, стабилизированное в `compose-material3` 1.4.x в 2026 году. Это не отдельная версия, а надстройка: вы можете использовать классические M3-компоненты и Expressive-компоненты в одном приложении. M3 Expressive добавляет:

- **MaterialExpressiveTheme** — новую тему-обёртку для активации Expressive-фич
- **MotionScheme** — систему motion-theming с двумя пресетами: `expressive` и `standard`
- **Расширенную библиотеку форм** (Expanded Shape Library)
- **Новые компоненты**: `LoadingIndicator`, `ContainedLoadingIndicator`, `SplitButton`, `FloatingActionButtonMenu`
- **Вибрантные цветовые схемы** — более насыщенные палитры

### MaterialExpressiveTheme — точка входа

```kotlin
import androidx.compose.material3.expressive.MaterialExpressiveTheme
import androidx.compose.material3.expressive.MotionScheme

@Composable
fun AppTheme(content: @Composable () -> Unit) {
    MaterialExpressiveTheme(
        colorScheme = if (isSystemInDarkTheme()) DarkColorScheme else LightColorScheme,
        motionScheme = MotionScheme.expressive(),  // или MotionScheme.standard()
        content = content
    )
}
```

### Расширенная система форм (Expanded Shape Library)

M3 Expressive вводит более широкую библиотеку форм: экстремальные скругления, асимметричные формы, формы «react-to-context»:

```kotlin
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.Shapes

val ExpressiveShapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(28.dp),
)

// Дополнительные Expressive-формы
val SquircleShape = RoundedCornerShape(50)  // процент → squircle-подобная форма
val AsymmetricShape = RoundedCornerShape(
    topStart = 16.dp,
    topEnd = 0.dp,
    bottomEnd = 16.dp,
    bottomStart = 0.dp
)
```

### Новые компоненты M3 Expressive

#### LoadingIndicator и ContainedLoadingIndicator

Новые индикаторы загрузки с motion physics и «attention-holding» поведением. В отличие от `CircularProgressIndicator`, который выглядит как крутящаяся палочка, `LoadingIndicator` использует физику пружины для создания более выразительной анимации.

```kotlin
import androidx.compose.material3.expressive.*

@Composable
fun LoadingExample(isLoading: Boolean) {
    if (isLoading) {
        LoadingIndicator(
            modifier = Modifier.padding(16.dp)
        )
    }
}

@Composable
fun ContainedLoadingExample() {
    // ContainedLoadingIndicator — версия внутри контейнера с границами
    ContainedLoadingIndicator(
        modifier = Modifier
            .fillMaxWidth()
            .height(48.dp)
            .padding(16.dp)
    )
}
```

#### SplitButton

Композитная кнопка с основным действием и вторичной «опциональной» частью (обычно меню опций):

```kotlin
import androidx.compose.material3.SplitButton
import androidx.compose.material3.SplitButtonDefaults

@Composable
fun SaveWithOptionsMenu(onSave: () -> Unit, onShowOptions: () -> Unit) {
    SplitButton(
        mainButton = {
            Button(onClick = onSave) { Text("Сохранить") }
        },
        secondaryButton = {
            SplitButtonDefaults.SecondaryButton(
                onClick = onShowOptions,
                content = {
                    Icon(Icons.Default.MoreVert, contentDescription = "Опции сохранения")
                }
            )
        }
    )
}
```

> **Когда использовать:** `SplitButton` идеален для действий с вариациями — «Сохранить» / «Сохранить как», «Отправить» / «Отправить копию», «Удалить» / «Удалить навсегда».

#### FloatingActionButtonMenu (FAB Menu)

Раскрытое меню из FAB, заменяет `speed dial` и наборы из нескольких stacked FAB:

```kotlin
import androidx.compose.material3.FloatingActionButtonMenu
import androidx.compose.material3.FloatingActionButtonMenuItem
import androidx.compose.material3.ExpandableFloatingActionButton

@Composable
fun FabMenuExample(
    expanded: Boolean,
    onToggle: () -> Unit,
    onCreate: () -> Unit,
    onShare: () -> Unit,
    onDelete: () -> Unit
) {
    FloatingActionButtonMenu(
        expandableFab = {
            ExpandableFloatingActionButton(
                onClick = onToggle,
                expanded = expanded
            )
        },
        expanded = expanded,
        content = {
            FloatingActionButtonMenuItem(
                onClick = onCreate,
                icon = { Icon(Icons.Default.Add, contentDescription = "Создать") },
                text = { Text("Создать") }
            )
            FloatingActionButtonMenuItem(
                onClick = onShare,
                icon = { Icon(Icons.Default.Share, contentDescription = "Поделиться") },
                text = { Text("Поделиться") }
            )
            FloatingActionButtonMenuItem(
                onClick = onDelete,
                icon = { Icon(Icons.Default.Delete, contentDescription = "Удалить") },
                text = { Text("Удалить") }
            )
        }
    )
}
```

> **Когда использовать:** FAB Menu рекомендуется вместо нескольких FAB, потому что Material Design спецификация говорит: на экране должен быть только один FAB. Если у вас несколько действий — используйте FAB Menu.

### Вибрантные цветовые схемы

M3 Expressive добавляет более насыщенные палитры для акцентных компонентов:

```kotlin
// Стандартная M3-палитра
val StandardScheme = lightColorScheme(
    primary = Color(0xFF6750A4),
    // ...
)

// Expressive — более насыщенные цвета
val VibrantScheme = lightColorScheme(
    primary = Color(0xFF8B4FFB),  // более яркий purple
    secondary = Color(0xFFFF5C8A), // более яркий pink
    tertiary = Color(0xFF00D4AA), // более яркий teal
    // ...
)
```

---

## 22.4. MotionScheme: motion physics

Главное нововведение M3 Expressive — **MotionScheme**: система motion-theming, аналогичная ColorScheme, но для анимаций. Определяет предустановленные `AnimationSpec` для различных категорий движения.

```kotlin
import androidx.compose.material3.expressive.MotionScheme

@Composable
fun MotionSchemeExample() {
    val motionScheme = MaterialTheme.motionScheme

    // Получаем предустановленные specs из текущей темы
    val defaultSpatialSpec = motionScheme.defaultSpatialSpec()
    val fastSpatialSpec = motionScheme.fastSpatialSpec()
    val defaultEffectsSpec = motionScheme.defaultEffectsSpec()
    val fastEffectsSpec = motionScheme.fastEffectsSpec()

    // Используем в анимациях
    val color by animateColorAsState(
        targetValue = if (isSelected) primary else surface,
        animationSpec = defaultEffectsSpec,
        label = "color"
    )

    val padding by animateDpAsState(
        targetValue = if (isSelected) 24.dp else 8.dp,
        animationSpec = defaultSpatialSpec,
        label = "padding"
    )
}
```

### Два пресета MotionScheme

| Пресет | Описание | Когда использовать |
|--------|----------|--------------------|
| `MotionScheme.expressive()` | Более живые, пружинистые анимации с заметными overshoot'ами | Рекомендуется Material для большинства приложений; особенно для «hero moments» — ключевых взаимодействий |
| `MotionScheme.standard()` | Более сдержанные, предсказуемые анимации | Корпоративные приложения, accessibility-focused UI, сценарии где важна предсказуемость |

### Motion tokens

Material 3 Expressive определяет motion tokens (на уровне дизайн-системы, не API):
- `md.sys.motion.spring.fast.spatial` — быстрое пространственное движение
- `md.sys.motion.spring.default.spatial` — стандартное пространственное движение
- `md.sys.motion.spring.fast.effects` — быстрые эффекты (color, opacity)
- `md.sys.motion.spring.default.effects` — стандартные эффекты

В Compose эти токены доступны через `MotionScheme` API — не нужно вручную настраивать spring-параметры.

> **Best practice:** используйте `MotionScheme.expressive()` по умолчанию для новых приложений. Он рекомендуется Material design team и обеспечивает консистентность движения во всём приложении.

---

## 22.5. Адаптивный дизайн и доступность

Material Design 3 в Compose обеспечивает встроенную поддержку **адаптивного дизайна** — макеты автоматически адаптируются к размеру экрана и классу устройства через `WindowSizeClass`:

| Компонент | Форм-фактор | Устройство |
|-----------|-------------|------------|
| `NavigationBar` | Compact (< 600 dp) | Телефоны в портретной ориентации |
| `NavigationRail` | Medium (600–840 dp) | Планшеты, телефоны в ландшафтной |
| `NavigationDrawer` | Expanded (> 840 dp) | Десктоп, большие планшеты |

```kotlin
import androidx.compose.material3.windowsizeclass.WindowSizeClass
import androidx.compose.material3.windowsizeclass.WindowWidthSizeClass

@Composable
fun AdaptiveApp(windowSizeClass: WindowSizeClass) {
    val showNavigationRail = windowSizeClass.widthSizeClass >= WindowWidthSizeClass.Medium

    Scaffold(
        bottomBar = {
            if (!showNavigationRail) {
                NavigationBar { /* items */ }
            }
        }
    ) { padding ->
        Row(modifier = Modifier.padding(padding)) {
            if (showNavigationRail) {
                NavigationRail { /* items */ }
            }
            // Основной контент
        }
    }
}
```

### Доступность (Accessibility)

В Compose Multiplatform 1.11.0 для iOS добавлена поддержка VoiceOver, AssistiveTouch и Full Keyboard Access. Общие принципы доступности включают:

- Достаточный контраст текста и фона (WCAG AA — минимум 4.5:1 для обычного текста, 3:1 для крупного).
- Семантические описания для screen readers:

```kotlin
import androidx.compose.ui.semantics.contentDescription
import androidx.compose.ui.semantics.semantics

@Composable
fun AccessibleImageButton(onClick: () -> Unit) {
    IconButton(
        onClick = onClick,
        modifier = Modifier.semantics {
            contentDescription = "Удалить задачу"
        }
    ) {
        Icon(Icons.Default.Delete, contentDescription = null)
    }
}
```

- Поддержка масштабирования текста (уважайте `LocalDensity` и `FontScale`).
- Минимальные размеры touch-target: 48×48 dp (Material guideline).
- Навигация с клавиатуры (важно для Desktop и Web).

> **Подробнее:** см. раздел [28. Доступность (Accessibility)](28-accessibility.md) — там разобраны Semantics API, скринридеры, контраст и тестирование a11y.

---

⬅️ [← Глубокое погружение в Compose](21-deep-dive-compose.md) | [К содержанию](README.md) | [Интеграция и лучшие практики →](23-integration-best-practices.md) ➡️
