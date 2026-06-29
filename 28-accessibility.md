# Доступность (Accessibility / a11y)

> Доступность — это практика создания приложений, которыми могут пользоваться люди с ограниченными возможностями: нарушениями зрения, слуха, моторики или когнитивных функций. В Compose Multiplatform доступность встроена в семантическую модель UI через API `semantics {}`. Согласно стандарту WCAG 2.1 (Web Content Accessibility Guidelines), приложение должно быть воспринимаемым, управляемым, понятным и устойчивым. Во многих странах доступность — юридическое требование (ADA в США, EAA в ЕС с 2025 года, ФЗ-419 в России для госсектора).

---

## 28.1. Compose Semantics API

Compose строит **семантическое дерево** (semantics tree) параллельно с деревом UI. Это дерево используется скринридерами (TalkBack на Android, VoiceOver на iOS) и службами доступности для описания элементов пользователю. Каждому `@Composable` можно добавить семантику через модификатор `semantics {}` или передавая параметры вроде `contentDescription`.

**Базовый пример:**
```kotlin
import androidx.compose.ui.semantics.contentDescription
import androidx.compose.ui.semantics.semantics
import androidx.compose.ui.semantics.stateDescription

@Composable
fun LikeButton(isLiked: Boolean, onClick: () -> Unit) {
    IconButton(
        onClick = onClick,
        modifier = Modifier.semantics {
            contentDescription = if (isLiked) "Убрать лайк" else "Поставить лайк"
            stateDescription = if (isLiked) "Нравится" else "Не нравится"
        }
    ) {
        Icon(
            imageVector = if (isLiked) Icons.Filled.Favorite else Icons.Outlined.FavoriteBorder,
            contentDescription = null // Описано в semantics
        )
    }
}
```

Семантическое дерево отличается от визуального. Например, иконка без текста невидима для скринридера, но с `contentDescription` или `semantics {}` она становится доступной. Всегда указывайте `contentDescription = null` для декоративных иконок, чтобы скринридер их не озвучивал дважды.

---

## 28.2. contentDescription и состояние элементов

Для стандартных M3-компонентов (Button, TextField, Checkbox) Compose автоматически предоставляет базовую семантику. Но для кастомных элементов и иконок необходимо указывать `contentDescription`.

**Правила contentDescription:**
```kotlin
// ✅ Правильно — информативное описание для интерактивного элемента
Icon(
    imageVector = Icons.Outlined.Delete,
    contentDescription = "Удалить задачу"
)

// ✅ Правильно — декоративная иконка не озвучивается
Icon(
    imageVector = Icons.Filled.Star,
    contentDescription = null
)

// ❌ Плохо — неинформативное описание
Icon(
    imageVector = Icons.Outlined.Delete,
    contentDescription = "Иконка" // Бесполезно для пользователя
)
```

**Состояния для переключателей:**
```kotlin
@Composable
fun ToggleSwitch(isEnabled: Boolean, onToggle: () -> Unit) {
    Switch(
        checked = isEnabled,
        onCheckedChange = { onToggle() },
        modifier = Modifier.semantics {
            stateDescription = if (isEnabled) "Включено" else "Выключено"
        }
    )
}
```

---

## 28.3. Объединение семантики (mergeDescendants)

Иногда элемент состоит из нескольких дочерних компонентов, но для скринридера должен озвучиваться как единое целое. Например, карточка с аватаром, именем и описанием — пользователь не должен навигировать по каждому внутреннему элементу отдельно.

```kotlin
import androidx.compose.ui.semantics.semantics
import androidx.compose.ui.semantics.mergeDescendants

@Composable
fun UserCard(user: User) {
    Card(
        modifier = Modifier.semantics(mergeDescendants = true) {
            contentDescription = "${user.name}, ${user.role}. ${user.status}"
        }
    ) {
        Row(modifier = Modifier.padding(16.dp)) {
            AsyncImage(
                model = user.avatarUrl,
                contentDescription = null, // Декоративное — описание в родителе
                modifier = Modifier.size(48.dp)
            )
            Spacer(modifier = Modifier.width(12.dp))
            Column {
                Text(user.name, fontWeight = FontWeight.Bold)
                Text(user.role, style = MaterialTheme.typography.bodySmall)
            }
        }
    }
}
```

Когда `mergeDescendants = true`, скринридер озвучит карточку как один элемент: «Алиса Иванова, разработчик. В сети». Без `mergeDescendants` пользователю пришлось бы отдельно навигировать к аватару, имени и роли.

---

## 28.4. Минимальный размер области нажатия

Согласно Material Design 3 и WCAG, минимальный размер области нажатия — **48 x 48 dp** (рекомендуемый — 48 dp, минимум допустимый — 44 dp на iOS). Если визуальный элемент меньше (например, иконка 24 dp), увеличивайте область нажатия через `Modifier.size(48.dp)` с `Alignment.Center`.

```kotlin
@Composable
fun SmallIconButton(onClick: () -> Unit) {
    // Визуально — 24 dp, область нажатия — 48 dp
    IconButton(
        onClick = onClick,
        modifier = Modifier.size(48.dp)
    ) {
        Icon(
            imageVector = Icons.Outlined.Settings,
            contentDescription = "Настройки",
            modifier = Modifier.size(24.dp)
        )
    }
}

// Для кастомных кликабельных элементов:
Box(
    modifier = Modifier
        .size(48.dp)
        .clickable { /* действие */ },
    contentAlignment = Alignment.Center
) {
    Icon(Icons.Outlined.Close, contentDescription = "Закрыть", modifier = Modifier.size(20.dp))
}
```

---

## 28.5. Контрастность цветов

WCAG 2.1 требует коэффициент контрастности не менее **4.5:1** для обычного текста и **3:1** для крупного текста (18pt+ или 14pt bold+). Material Design 3 автоматически создаёт контрастные цветовые схемы через tone-based палитру, но при кастомизации цветов проверяйте контраст.

```kotlin
// Проверка контрастности вручную (утилита):
fun contrastRatio(color1: Color, color2: Color): Float {
    val luminance1 = calculateRelativeLuminance(color1)
    val luminance2 = calculateRelativeLuminance(color2)
    return (maxOf(luminance1, luminance2) + 0.05) /
           (minOf(luminance1, luminance2) + 0.05)
}

// Использование в Preview:
@Preview(showBackground = true)
@Composable
fun ContrastPreview() {
    val onSurface = MaterialTheme.colorScheme.onSurface
    val surface = MaterialTheme.colorScheme.surface
    Text(
        text = "Контраст: ${"%.1f".format(contrastRatio(onSurface, surface))}:1",
        color = if (contrastRatio(onSurface, surface) >= 4.5f) Color.Green else Color.Red
    )
}
```

Для динамического цвета (Dynamic Color на Android 12+) система автоматически подбирает контрастные цвета, но на iOS и Desktop нужно убедиться, что ваша цветовая схема соответствует требованиям.

---

## 28.6. Масштабирование шрифтов и крупный текст

Пользователи могут увеличить системный размер шрифта в настройках устройства. Compose автоматически реагирует на системные настройки через `TextFlow` и `LocalTextStyle`. Однако фиксированные размеры в dp (высота контейнеров, отступы) могут не подстроиться.

**Правила для масштабируемого текста:**
```kotlin
@Composable
fun ScalableTaskItem(task: Task) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .heightIn(min = 48.dp) // Минимальная высота, а не фиксированная
    ) {
        Text(
            text = task.title,
            style = MaterialTheme.typography.bodyLarge,
            maxLines = 2, // Ограничьте строки, чтобы текст не ломал лейаут
            overflow = TextOverflow.Ellipsis
        )
    }
}
```

Используйте `sp` для размеров текста (Compose делает это по умолчанию через `TextStyle`), а не `dp`. Избегайте `heightIn(max = ...)` для текстовых контейнеров — при увеличенном шрифте текст может не поместиться.

---

## 28.7. Управление фокусом и клавиатурная навигация

Для Desktop и Web (Wasm) клавиатурная навигация критична. Compose поддерживает фокус через `FocusRequester` и порядок табуляции.

```kotlin
import androidx.compose.ui.focus.FocusRequester
import androidx.compose.ui.focus.focusRequester

@Composable
fun LoginForm() {
    val emailFocus = FocusRequester()
    val passwordFocus = FocusRequester()

    Column(modifier = Modifier.padding(16.dp)) {
        OutlinedTextField(
            value = email,
            onValueChange = { email = it },
            label = { Text("Email") },
            modifier = Modifier.focusRequester(emailFocus),
            keyboardOptions = KeyboardOptions(imeAction = ImeAction.Next)
        )
        Spacer(modifier = Modifier.height(8.dp))
        OutlinedTextField(
            value = password,
            onValueChange = { password = it },
            label = { Text("Пароль") },
            modifier = Modifier.focusRequester(passwordFocus),
            keyboardOptions = KeyboardOptions(imeAction = ImeAction.Done)
        )
    }

    LaunchedEffect(Unit) {
        emailFocus.requestFocus() // Автофокус на поле email
    }
}
```

---

## 28.8. Тестирование доступности

Compose Testing позволяет проверять семантику в тестах, что особенно полезно в CI.

```kotlin
import androidx.compose.ui.semantics.SemanticsProperties
import androidx.compose.ui.test.*

class AccessibilityTest {
    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `delete button has content description`() {
        composeTestRule.setContent { DeleteButton(onClick = {}) }
        composeTestRule
            .onNodeWithContentDescription("Удалить задачу")
            .assertExists()
    }

    @Test
    fun `decorative icon has no content description`() {
        composeTestRule.setContent { DecorativeStar() }
        composeTestRule
            .onNode(hasContentDescription())
            .assertDoesNotExist()
    }

    @Test
    fun `card merges child semantics`() {
        composeTestRule.setContent { UserCard(User("Алиса", "Разработчик")) }
        composeTestRule
            .onNodeWithContentDescription("Алиса, разработчик")
            .assertIsDisplayed()
    }
}
```

---

## 28.9. Чеклист доступности

| Проверка | Как проверить |
|----------|---------------|
| Все интерактивные элементы имеют `contentDescription` | Запустить TalkBack/VoiceOver и проверить озвучивание |
| Декоративные иконки имеют `contentDescription = null` | Включить скринридер — не должны озвучиваться |
| Область нажатия минимум 48x48 dp | Измерить через Layout Inspector |
| Контраст текста ≥ 4.5:1 | Использовать Color Contrast Checker или формулу выше |
| Карточки используют `mergeDescendants = true` | Проверить в скринридере — карточка озвучивается как единый элемент |
| Текст масштабируется без обрезки | Увеличить шрифт в настройках до максимального |
| Клавиатурная навигация работает (Desktop/Web) | Пройти все элементы через Tab |
| Фокус виден на всех платформах | Визуальная проверка фокусного кольца |

---

⬅️ [← Quick Reference](31-quick-reference.md) | [К содержанию](README.md) | [Анимации и Motion →](29-animations-motion.md) ➡️
