# Material Design 3 (MD3) в Compose

> Material Design 3 — обновлённая дизайн-система от Google, интегрированная в Compose. Предоставляет набор компонентов, цветов, типографики и форм для создания современных пользовательских интерфейсов.

---

## 3.1. Подключение зависимостей

Добавьте в `build.gradle.kts`:
```kotlin
implementation(compose.material3)
```

Для использования Material Design 3 в Compose Multiplatform достаточно добавить одну зависимость. Библиотека `compose.material3` включает в себя все основные компоненты, цветовые схемы, типографику и систему форм. Убедитесь, что вы используете актуальную версию Compose, которая поддерживает MD3 — начиная с Compose 1.1.0 Material 3 доступен как стабильная зависимость.

---

## 3.2. Темы и типографика

Темы определяют цвета, типографику и формы.

**Создание темы:**
```kotlin
@Composable
fun MyAppTheme(content: @Composable () -> Unit) {
    MaterialTheme(
        colorScheme = lightColorScheme(),
        typography = Typography,
        content = content
    )
}
```

**Настройка цветов:**
```kotlin
val colorScheme = lightColorScheme(
    primary = Color(0xFF6200EE),
    secondary = Color(0xFF0175FF),
    tertiary = Color(0xFFFF4081)
)
```

Тема в MD3 строится вокруг концепции «динамических цветов» (Dynamic Color), которая автоматически адаптирует палитру приложения на основе обоев пользователя (на Android 12+). Если динамические цвета недоступны, используется явно заданная цветовая схема. Цветовая схема MD3 включает более 50 цветовых токенов, покрывающих все состояния компонентов — от основного фона до цвета текста на отключённых кнопках.

---

## 3.3. Основные компоненты

- **Кнопки:** `Button`, `FloatingActionButton`.
- **Текстовые поля:** `TextField`.
- **Карточки:** `Card`.
- **Диалоги:** `AlertDialog`.
- **Списки:** `LazyColumn`, `LazyRow`.

**Пример кнопки:**
```kotlin
Button(onClick = {}) {
    Text("Press me")
}
```

**Пример карточки:**
```kotlin
Card(
    modifier = Modifier.padding(8.dp),
    elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Заголовок", style = MaterialTheme.typography.titleLarge)
        Text("Описание карточки", style = MaterialTheme.typography.bodyMedium)
    }
}
```

**Пример текстового поля:**
```kotlin
var text by remember { mutableStateOf("") }
TextField(
    value = text,
    onValueChange = { text = it },
    label = { Text("Введите текст") },
    modifier = Modifier.fillMaxWidth()
)
```

---

## 3.4. Адаптивные UI

MD3 поддерживает адаптивный дизайн:
- `provideAdaptivePlatformWindowInsets` — для корректных отступов.
- `Scaffold` — базовый макет с панелью приложений, плавающей кнопкой и т. д.

**Пример макета:**
```kotlin
Scaffold(
    topBar = { TopAppBar(title = { Text("My App") }) },
    floatingActionButton = { FloatingActionButton(onClick = {}) { Icon(Icons.Filled.Add, "add") } }
) { innerPadding ->
    LazyColumn(modifier = Modifier.padding(innerPadding)) {
        items(items = listOf("Item 1", "Item 2")) { item ->
            Text(item, style = MaterialTheme.typography.bodyLarge)
        }
    }
}
```

`Scaffold` — основной строительный блок для экранов в MD3. Он предоставляет слоты для `TopAppBar`, `BottomAppBar`, `FloatingActionButton`, `Snackbar` и основного контента. Компонент автоматически вычисляет отступы для контента, чтобы он не перекрывался системными элементами (статус-бар, навигационная панель).

---

## 3.5. Расширенные компоненты MD3

- **BottomAppBar** — нижняя панель с иконками.
- **Snackbar** — всплывающие уведомления.
- **ProgressIndicator** — индикаторы загрузки.
- **TopAppBar** с меню и навигацией.

**Пример Snackbar:**
```kotlin
var visible by remember { mutableStateOf(false) }
val coroutineScope = rememberCoroutineScope()

Scaffold(
    snackbarHost = { SnackbarHost(...) }
) { contentPadding ->
    Button(onClick = { coroutineScope.launch { visible = true } }) {
        Text("Show Snackbar")
    }
}
```

**Пример BottomAppBar:**
```kotlin
Scaffold(
    bottomBar = {
        BottomAppBar {
            IconButton(onClick = {}) { Icon(Icons.Filled.Home, "Home") }
            IconButton(onClick = {}) { Icon(Icons.Filled.Search, "Search") }
            IconButton(onClick = {}) { Icon(Icons.Filled.Settings, "Settings") }
        }
    }
) { innerPadding ->
    // Основной контент
}
```

---

## 3.6. Кастомизация тем

Создайте кастомные цвета, типографику, формы:
```kotlin
val customTypography = Typography(
    bodyLarge = TextStyle(
        fontFamily = FontFamily.Serif,
        fontSize = 18.sp
    )
)

val customColorScheme = darkColorScheme(
    primary = Color(0xFF37474F),
    onPrimary = Color(0xFFF5F5F5)
)
```

Кастомизация тем в MD3 выходит далеко за рамки простого изменения цветов. Вы можете определить собственную типографику с разными шрифтами для заголовков и основного текста, настроить формы (rounded, cut) для всех компонентов, а также создать полностью кастомные компоненты, которые следуют принципам дизайн-системы. Это позволяет поддерживать единый визуальный стиль во всём приложении.

---

⬅️ [← Compose Multiplatform](02-compose-multiplatform.md) | [К содержанию](README.md) | [Практический пример →](04-practical-example.md) ➡️
