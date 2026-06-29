# Material Design 3: глубокое погружение

> Детальное рассмотрение дизайн-токенов, компонентной библиотеки, M3 Expressive и адаптивного дизайна — с примерами кода для Compose Multiplatform.

---

## 22.1. Дизайн-токены и система тем

Material Design 3 основан на системе **дизайн-токенов** — семантически именованных значений, которые определяют внешний вид приложения. В Compose M3 эти токены представлены объектами `ColorScheme`, `Typography` и `Shape`:

```kotlin
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
    // ...
)

// Применение темы
MaterialTheme(
    colorScheme = if (isSystemInDarkTheme()) DarkColorScheme else LightColorScheme,
    typography = Typography,
    shapes = Shapes,
    content = { /* ваш UI */ }
)
```

Система M3 определяет **семантические цветовые роли**: `primary` (основной цвет), `secondary` (дополнительный), `tertiary` (третий акцент), `error` (ошибки), `surface` (поверхности), `background` (фон) — и их варианты `on*` (для текста/иконок поверх) и `*Container` (для контейнеров). Это позволяет адаптировать палитру под бренд, не меняя семантику цветов в коде.

**Динамический цвет** (Dynamic Color) на Android 12+ автоматически генерирует цветовую схему на основе обоев пользователя. В Compose Multiplatform динамический цвет работает на Android, а для других платформ используются статические схемы.

---

## 22.2. Библиотека основных компонентов

Material Design 3 в Compose предоставляет обширный набор компонентов, организованных по категориям:

### Кнопки

`Button`, `FilledTonalButton`, `OutlinedButton`, `TextButton`, `ElevatedButton`, `FilledIconButton`, `OutlinedIconButton`, `IconButton`, `FloatingActionButton`, `ExtendedFloatingActionButton`

### Контейнеры

`Card`, `ElevatedCard`, `FilledCard`, `OutlinedCard`, `Surface`, `Scaffold`

### Навигация

`NavigationBar`, `NavigationRail`, `NavigationDrawer`, `NavigationItem`, `TabRow`, `ScrollableTabRow`

### Диалоги и окна

`AlertDialog`, `ModalBottomSheet`, `DatePicker`, `TimePicker`

### Ввод данных

`TextField`, `OutlinedTextField`, `FilledTextField`, `PasswordField`, `DropdownMenu`, `ExposedDropdownMenuBox`, `Slider`, `Switch`, `Checkbox`, `RadioButton`

### Отображение данных

`ListItem`, `Divider`, `ProgressIndicator`, `CircularProgressIndicator`, `LinearProgressIndicator`, `Badge`, `AssistChip`, `FilterChip`, `SuggestionChip`, `InputChip`

### Типографика

M3 определяет шкалу типографики с ролями: `displayLarge`, `displayMedium`, `displaySmall`, `headlineLarge`, `headlineMedium`, `headlineSmall`, `titleLarge`, `titleMedium`, `titleSmall`, `bodyLarge`, `bodyMedium`, `bodySmall`, `labelLarge`, `labelMedium`, `labelSmall`.

Каждый компонент M3 в Compose следует принципам **elevation** через систему **tonal elevation** — вместо традиционных теней M3 использует смешивание цвета поверхности с цветом `primary`, создавая более мягкий и современный визуальный эффект.

---

## 22.3. M3 Expressive: новейшая эволюция

Material 3 Expressive — расширение Material Design 3, включающее ключевые обновления:

### Расширенная система форм (Expanded Shape Library)

M3 Expressive вводит более широкую библиотеку форм, включая экстремальные скругления, асимметричные формы и формы, реагирующие на контекст:

```kotlin
// M3 Expressive shapes
val ExpressiveShapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(28.dp),
)
```

### Система motion-физики

Анимации в M3 Expressive основаны на физической модели, обеспечивающей естественное движение. В Compose это реализуется через `spring()`-анимации с настраиваемыми параметрами damping и stiffness.

### Визуально акцентированная типографика

M3 Expressive расширяет типографическую шкалу, позволяя создавать более выразительные иерархии текста. Поддерживаются пользовательские шрифтовые семьи и весовые вариации.

### Вибрантные цветовые схемы

Новые цветовые схемы с более насыщенными и яркими цветами, адаптирующиеся к контексту использования. В Compose Material 3 Expressive дополняет Android 16 visual style и system UI.

---

## 22.4. Адаптивный дизайн и доступность

Material Design 3 в Compose обеспечивает встроенную поддержку **адаптивного дизайна** — макеты автоматически адаптируются к размеру экрана и классу устройства:

| Компонент | Форм-фактор | Устройство |
|-----------|-------------|------------|
| `NavigationBar` | Компактный экран | Телефоны |
| `NavigationRail` | Средний экран | Планшеты |
| `NavigationDrawer` | Большой экран | Десктоп |

**Доступность (Accessibility)** является приоритетом M3. В Compose Multiplatform 1.8.0 для iOS добавлена поддержка VoiceOver, AssistiveTouch и Full Keyboard Access. Общие принципы доступности включают: достаточный контраст текста и фона (WCAG AA), семантические описания для screen readers (`semantics { contentDescription = "..." }`), поддержку масштабирования текста и навигацию с клавиатуры.

---

⬅️ [← Глубокое погружение в Compose](21-deep-dive-compose.md) | [К содержанию](README.md) | [Интеграция и лучшие практики →](23-integration-best-practices.md) ➡️
