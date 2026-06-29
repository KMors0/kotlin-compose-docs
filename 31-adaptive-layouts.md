# Адаптивные лейауты и foldables

> Адаптивный дизайн — это не «сделать приложение отзывчивым». Это проектирование интерфейса, который оптимально использует пространство на любом устройстве: от узкого телефона до широкоэкранного планшета, от раскладного смартфона до десктопа с двумя мониторами. Material Design 3 определяет три категории размера окна: Compact, Medium и Expanded.

---

## 31.1. WindowSizeClass

`WindowSizeClass` — центральный API адаптивности в Compose. Он классифицирует ширину окна в одну из трёх категорий:

| Категория | Ширина | Типичные устройства |
|-----------|--------|---------------------|
| Compact | < 600 dp | Телефон в портретной ориентации |
| Medium | 600–839 dp | Телефон в ландшафтной, планшет в портретной, складной телефон в развёрнутом виде |
| Expanded | >= 840 dp | Планшет в ландшафтной, десктоп |

```kotlin
import androidx.compose.material3.windowsizeclass.*

@Composable
fun AdaptiveApp() {
    val windowSizeClass = calculateWindowSizeClass(activity = LocalContext.current as Activity)
    val widthSizeClass = windowSizeClass.widthSizeClass

    when (widthSizeClass) {
        WindowWidthSizeClass.Compact -> CompactLayout()
        WindowWidthSizeClass.Medium -> MediumLayout()
        WindowWidthSizeClass.Expanded -> ExpandedLayout()
    }
}
```

> **KMP:** На iOS и Desktop `WindowSizeClass` вычисляется через `expect/actual`. Для Compose Multiplatform используйте `calculateWindowSizeClass()` из `compose-material3-adaptive`.

---

## 31.2. Навигационные компоненты: NavigationBar, NavigationRail, NavigationDrawer

Material 3 определяет три навигационных компонента, каждый для своей категории размера:

```kotlin
import androidx.compose.material3.*

@Composable
fun AdaptiveNavigation(
    windowWidthSizeClass: WindowWidthSizeClass,
    selectedTab: Int,
    onTabSelected: (Int) -> Unit,
    content: @Composable () -> Unit
) {
    when (windowWidthSizeClass) {
        WindowWidthSizeClass.Compact -> {
            // Телефон: нижний бар
            Scaffold(
                bottomBar = {
                    NavigationBar {
                        items.forEachIndexed { index, item ->
                            NavigationBarItem(
                                icon = { Icon(item.icon, contentDescription = null) },
                                label = { Text(item.label) },
                                selected = selectedTab == index,
                                onClick = { onTabSelected(index) }
                            )
                        }
                    }
                }
            ) { content() }
        }
        WindowWidthSizeClass.Medium -> {
            // Планшет / раскладной: боковая панель
            PermanentNavigationDrawer(drawerContent = {
                NavigationDrawerItem(
                    icon = { Icon(Icons.Default.Home, null) },
                    label = { Text("Home") },
                    selected = selectedTab == 0,
                    onClick = { onTabSelected(0) }
                )
                // ...
            }) { content() }
        }
        WindowWidthSizeClass.Expanded -> {
            // Десктоп: навигационный rail
            NavigationRail {
                items.forEachIndexed { index, item ->
                    NavigationRailItem(
                        icon = { Icon(item.icon, contentDescription = null) },
                        label = { Text(item.label) },
                        selected = selectedTab == index,
                        onClick = { onTabSelected(index) }
                    )
                }
            }
            Box(modifier = Modifier.padding(start = 80.dp)) { content() }
        }
    }
}
```

---

## 31.3. ListDetailPaneScaffold

`ListDetailPaneScaffold` — готовый компонент для master-detail паттерна (список + детали). На Compact-размере это два экрана (с навигацией), на Medium/Expanded — дваpane рядом.

```kotlin
import androidx.compose.material3.adaptive.*

@OptIn(ExperimentalMaterial3AdaptiveApi::class)
@Composable
fun TasksPaneScaffold() {
    val navigator = rememberListDetailPaneScaffoldNavigator()

    ListDetailPaneScaffold(
        directive = navigator.scaffoldDirective,
        value = navigator.scaffoldValue,
        listPane = {
            LazyColumn {
                items(tasks) { task ->
                    ListItem(
                headlineContent = { Text(task.title) },
                modifier = Modifier.clickable {
                    navigator.navigateTo(TaskDetailPane(task.id))
                }
                    )
                }
            }
        },
        detailPane = {
            navigator.currentDestination?.let { destination ->
                TaskDetailScreen(taskId = (destination as TaskDetailPane).taskId)
            }
        }
    )
}
```

---

## 31.4. Foldables (раскладные устройства)

Samsung Galaxy Z Fold, Pixel Fold и аналоги предоставляют два экрана: компактный (складной) и раскладной (tabletop). Compose Multiplatform поддерживает foldables через `WindowSizeClass` и ` FoldingFeature`.

**Определение fold-state через expect/actual:**
```kotlin
// commonMain
expect enum class FoldState { Flat, Folded, HalfFolded }
expect fun getFoldState(): FoldState

// androidMain
actual enum class FoldState { Flat, Folded, HalfFolded }
actual fun getFoldState(): FoldState {
    val windowManager = context.getSystemService(Context.WINDOW_SERVICE) as WindowManager
    val foldingFeature = windowManager.getCurrentFoldState()
    return when {
        foldingFeature?.isFlat == true -> FoldState.Flat
        foldingFeature?.state == FoldingFeature.State.HALF_OPENED -> FoldState.HalfFolded
        else -> FoldState.Folded
    }
}
```

**Адаптация UI для HalfFolded (tabletop mode):**
```kotlin
@Composable
fun VideoPlayerLayout(foldState: FoldState) {
    if (foldState == FoldState.HalfFolded) {
        // Верхняя половина — видео, нижняя — управление
        Column(modifier = Modifier.fillMaxSize()) {
            Box(modifier = Modifier.weight(1f)) { VideoPlayer() }
            Box(modifier = Modifier.weight(1f)) { PlaybackControls() }
        }
    } else {
        // Обычный лейаут
        Column {
            VideoPlayer(modifier = Modifier.aspectRatio(16f/9f))
            PlaybackControls()
        }
    }
}
```

---

## 31.5. BoxWithConstraints и custom breakpoints

Для finer-grained адаптивности, когда трёх категорий WindowSizeClass недостаточно:

```kotlin
@Composable
fun AdaptiveGrid(items: List<Item>) {
    BoxWithConstraints {
        val columns = when {
            maxWidth < 400.dp -> 1
            maxWidth < 600.dp -> 2
            maxWidth < 840.dp -> 3
            else -> 4
        }
        LazyVerticalGrid(
            columns = GridCells.Fixed(columns),
            modifier = Modifier.fillMaxSize()
        ) {
            items(items) { item -> ItemCard(item) }
        }
    }
}
```

> **Предпочитайте `WindowSizeClass`** для основных layout-решений, а `BoxWithConstraints` — для тонкой настройки внутри компонентов.

---

⬅️ [← KMP Gotchas](34-kmp-gotchas.md) | [К содержанию](README.md) | [Оптимизация размера →](37-app-size-optimization.md) ➡️
