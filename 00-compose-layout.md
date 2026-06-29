# Как собирать интерфейс в Compose: гайд для тех, кто не понимает отступы

> Если ты пришёл из Flutter (или вообще без опыта UI) и не понимаешь, как расставить отступы, почему кнопка уехала, и как вообще думать о лейауте — этот гайд для тебя. Мы разберём визуальное мышление Compose от нуля, с кучей схем и примеров из мессенджера.

---

## Содержание

- [Главная идея: всё — прямоугольники](#главная-идея-всё--прямоугольники)
- [Modifier: как менять вид и положение](#modifier-как-менять-вид-и-положение)
- [Column: вертикальная стопка](#column-вертикальная-стопка)
- [Row: горизонтальный ряд](#row-горизонтальный-ряд)
- [Box: наложение друг на друга](#box-наложение-друг-на-друга)
- [padding vs margin: в чём разница?](#padding-vs-margin-в-чём-разница)
- [Как думать о лейауте: метод «от большого к малому»](#как-думать-о-лейауте-метод-от-большого-к-малому)
- [Разбираем экран чата](#разбираем-экран-чата)
- [Разбираем пузырь сообщения](#разбираем-пузырь-сообщения)
- [Разбираем список контактов](#разбираем-список-контактов)
- [Spacer: невидимый разделитель](#spacer-невидимый-разделитель)
- [fillMaxSize, fillMaxWidth, wrapContent: размеры](#fillmaxsize-fillmaxwidth-wrapcontent-размеры)
- [Weight: как делить пространство](#weight-как-делить-пространство)
- [Alignment: выравнивание внутри контейнера](#alignment-выравнивание-внутри-контейнера)
- [Scroll: прокрутка](#scroll-прокрутка)
- [Сравнение с Flutter](#сравнение-с-flutter)
- [Шпаргалка по Modifier](#шпаргалка-по-modifier)
- [Чеклист: почему элемент не там, где надо?](#чеклист-почему-элемент-не-там-где-надо)

---

## Главная идея: всё — прямоугольники

В Compose (как и в любом UI-фреймворке) каждый элемент интерфейса — это **прямоугольник**. Кнопка — прямоугольник. Текст — прямоугольник. Картинка — прямоугольник. Даже круглый аватар — это прямоугольник, в котором нарисован круг.

Твоя задача как разработчика UI — **расставить прямоугольники на экране**. Для этого нужно ответить на три вопроса для каждого прямоугольника:

1. **Какого он размера?** (ширина × высота)
2. **Где он находится?** (позиция относительно родителя)
3. **Что внутри?** (текст, иконка, другой прямоугольник)

Compose предлагает три основных способа расположить прямоугольники друг относительно друга: **Column** (вертикально), **Row** (горизонтально), **Box** (один поверх другого). Всё остальное — комбинации этих трёх.

---

## Modifier: как менять вид и положение

**Modifier** — это набор настроек, которые ты «навешиваешь» на элемент. Это центральный инструмент для лейаута. Аналогия: Modifier — как одежду, которую ты надеваешь на человека. Сам человек (Composable) — его тело, а Modifier — куртка, шапка, перчатки. Они не меняют человека, но меняют его внешний вид и поведение.

```kotlin
Text(
    text = "Привет",
    modifier = Modifier
        .padding(16.dp)        // Отступы вокруг текста
        .background(Color.Blue) // Фон
        .fillMaxWidth()         // Занять всю ширину
)
```

### Порядок Modifier'ов ВАЖЕН!

Это **самая частая ошибка новичков**. Modifier'ы применяются последовательно, и порядок меняет результат:

```kotlin
// Вариант 1: СНАЧАЛА padding, ПОТОМ background
Modifier
    .padding(16.dp)      // ← Отступ 16dp вокруг
    .background(Color.Blue) // ← Синий фон ЗАНИМАЕТ всё оставшееся пространство

// Результат: синий фон с отступом 16dp от краёв экрана
// [  16dp  ][======= СИНИЙ =======][  16dp  ]


// Вариант 2: СНАЧАЛА background, ПОТОМ padding
Modifier
    .background(Color.Blue) // ← Синий фон на весь размер
    .padding(16.dp)          // ← Отступ ВНУТРИ синего фона

// Результат: синий фон на всю ширину, текст сдвинут на 16dp внутрь
// [========== СИНИЙ ==========]
// [  16dp  ][Текст][  16dp  ]
```

**Правило:** Modifier'ы применяются «снаружи внутрь». Каждый следующий Modifier работает с тем, что осталось после предыдущего.

---

## Column: вертикальная стопка

`Column` — располагает дочерние элементы **сверху вниз**, один под другим.

```
┌─────────────────┐
│   Элемент 1     │
├─────────────────┤
│   Элемент 2     │
├─────────────────┤
│   Элемент 3     │
└─────────────────┘
```

```kotlin
Column {
    Text("Строка 1")
    Text("Строка 2")
    Text("Строка 3")
}
```

### Отступы между элементами: Arrangement

```kotlin
Column(
    verticalArrangement = Arrangement.spacedBy(8.dp)  // 8dp между элементами
) {
    Text("Строка 1")  // ← 8dp →
    Text("Строка 2")  // ← 8dp →
    Text("Строка 3")
}
```

Это УДОБНЕЕ, чем ставить padding на каждый элемент вручную. В Flutter для этого нужен `SizedBox(height: 8)` между каждым виджетом — здесь одна настройка на весь Column.

### Выравнивание по горизонтали: horizontalAlignment

```kotlin
Column(
    horizontalAlignment = Alignment.Start  // Все элементы прижаты влево
) {
    Text("Короткий")
    Text("Текст подлиннее")
    Text("Совсем длинный текст")
}

// Результат:
// Короткий
// Текст подлиннее
// Совсем длинный текст
// ↑ Всё выровнено по левому краю

Column(
    horizontalAlignment = Alignment.CenterHorizontally  // Все по центру
) {
    Text("Короткий")
    Text("Текст подлиннее")
}
//    Короткий
// Текст подлиннее
// ↑ Всё по центру
```

---

## Row: горизонтальный ряд

`Row` — располагает дочерние элементы **слева направо**, в один ряд.

```
┌──────┬──────┬──────┐
│ Эл.1 │ Эл.2 │ Эл.3 │
└──────┴──────┴──────┘
```

```kotlin
Row {
    Text("Имя:")
    Text("Алиса")
}
// Имя: Алиса
```

### Отступы между элементами: Arrangement

```kotlin
Row(
    horizontalArrangement = Arrangement.spacedBy(12.dp)  // 12dp между элементами
) {
    Icon(Icons.Default.Person, "Пользователь")  // 👤
    Text("Алиса")                                // Алиса
    Text("онлайн")                               // онлайн
}
// 👤 ←12dp→ Алиса ←12dp→ онлайн
```

### Выравнивание по вертикали: verticalAlignment

```kotlin
Row(
    verticalAlignment = Alignment.CenterVertically  // Элементы по центру по высоте
) {
    Icon(Icons.Default.Person, "Пользователь")  // Иконка 24dp
    Text("Алиса")                                // Текст 16dp
}
// Без CenterVertically иконка была бы выше текста
// С CenterVertically — они на одной линии по центру
```

---

## Box: наложение друг на друга

`Box` — располагает дочерние элементы **один поверх другого** (как стопка бумаги). Последний элемент — наверху.

```
┌─────────────────┐
│ ┌─────────────┐ │
│ │  Элемент 2  │ │  ← поверх Элемента 1
│ └─────────────┘ │
│   Элемент 1     │  ← на заднем плане
└─────────────────┘
```

```kotlin
Box {
    // Задний план: фоновая картинка
    Image(painterResource("bg.jpg"), contentDescription = null)

    // Передний план: текст поверх картинки
    Text(
        text = "ReMatch Chat",
        modifier = Modifier
            .align(Alignment.BottomStart)  // Прижать к левому нижнему углу
            .padding(16.dp)
    )
}
```

Box часто используется для:
- Текст поверх картинки (аватар + статус онлайн)
- Индикатор загрузки поверх контента
- FAB (кнопка) в углу экрана

---

## padding vs margin: в чём разница?

Во Flutter/HTML есть `margin` (отступ снаружи) и `padding` (отступ внутри). В Compose **есть только `padding`**, но порядок Modifier'ов определяет, что это — margin или padding:

```kotlin
// «Margin» — отступ СНАРУЖИ (между этим элементом и соседями)
Modifier
    .padding(16.dp)       // ← Это «margin»: пространство вокруг элемента
    .background(Color.Blue)

// «Padding» — отступ ВНУТРИ (между границей фона и содержимым)
Modifier
    .background(Color.Blue)
    .padding(16.dp)       // ← Это «padding»: пространство внутри синего фона
```

Визуально:

```
MARGIN (padding → background):       PADDING (background → padding):
┌─────────────────────────┐          ┌─────────────────────────┐
│  (пусто 16dp)           │          │ ███████████████████████ │
│  ┌───────────────────┐  │          │ ████ (пусто 16dp) ████ │
│  │ СИНИЙ ФОН + Текст │  │          │ ████ Текст        ████ │
│  └───────────────────┘  │          │ ████ (пусто 16dp) ████ │
│  (пусто 16dp)           │          │ ███████████████████████ │
└─────────────────────────┘          └─────────────────────────┘
```

**Шпаргалка:**
- Хочешь отступ **вокруг** элемента (как margin)? → `padding` перед `background`
- Хочешь отступ **внутри** элемента (между фоном и текстом)? → `padding` после `background`
- Нужны оба? → Два padding'а: `Modifier.padding(8.dp).background(Blue).padding(16.dp)`

---

## Как думать о лейауте: метод «от большого к малому»

Главный принцип: **начинай с самого большого контейнера и разбивай его на части**. Не думай сразу о каждом пикселе — думай блоками.

### Пример: экран мессенджера

```
┌──────────────────────────┐
│      Аппбар (Шапка)      │  ← Верхняя полоса с названием
├──────────────────────────┤
│                          │
│     Список сообщений     │  ← Занимает основную площадь
│     (прокручивается)     │
│                          │
├──────────────────────────┤
│  [Поле ввода] [Отправить]│  ← Нижняя панель
└──────────────────────────┘
```

**Как это перевести в Column:**

1. Самый большой контейнер — **Column** (всё вертикально: шапка → сообщения → ввод)
2. Внутри три элемента: TopAppBar, LazyColumn (сообщения), Row (поле + кнопка)

```kotlin
@Composable
fun ChatScreen() {
    Column(modifier = Modifier.fillMaxSize()) {
        // 1. Шапка
        ChatTopBar()

        // 2. Список сообщений — занимает всё свободное пространство
        LazyColumn(
            modifier = Modifier.weight(1f),  // ← Растягивается
            verticalArrangement = Arrangement.spacedBy(4.dp)
        ) {
            items(messages) { message ->
                MessageBubble(message)
            }
        }

        // 3. Нижняя панель
        MessageInputBar()
    }
}
```

---

## Разбираем экран чата

Давай соберём шапку чата (TopAppBar) по шагам:

```
┌────────────────────────────────────┐
│ ← [Аватар] Алиса          📞 ⋮   │
└────────────────────────────────────┘
```

**Шаг 1:** Вижу горизонтальный ряд → **Row**

**Шаг 2:** Разбиваю на части:
- Кнопка «назад» (←)
- Аватар
- Имя (Алиса)
- Пустое пространство (растягивается)
- Кнопка звонка (📞)
- Кнопка меню (⋮)

**Шаг 3:** Пишу код:

```kotlin
@Composable
fun ChatTopBar() {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 8.dp, vertical = 4.dp),
        verticalAlignment = Alignment.CenterVertically  // Всё по центру по высоте
    ) {
        // Кнопка «назад»
        IconButton(onClick = { /* навигация назад */ }) {
            Icon(Icons.Default.ArrowBack, "Назад")
        }

        // Аватар
        Box(
            modifier = Modifier
                .size(40.dp)
                .clip(CircleShape)
                .background(Color.Gray)
        ) {
            Text("А", modifier = Modifier.align(Alignment.Center))
        }

        Spacer(modifier = Modifier.width(8.dp))  // Отступ между аватаром и именем

        // Имя
        Text(
            text = "Алиса",
            style = MaterialTheme.typography.titleMedium,
            modifier = Modifier.weight(1f)  // ← Занимает всё свободное пространство
        )

        // Кнопки справа
        IconButton(onClick = { /* звонок */ }) {
            Icon(Icons.Default.Call, "Звонок")
        }
        IconButton(onClick = { /* меню */ }) {
            Icon(Icons.Default.MoreVert, "Меню")
        }
    }
}
```

**Ключевой момент:** `Modifier.weight(1f)` на имени означает «займи всё свободное пространство». Благодаря этому кнопки звонка и меню прижаты вправо.

---

## Разбираем пузырь сообщения

```
                    ┌─────────────────────┐
                    │ Привет! Как дела?   │
                    │              12:30  │
                    └─────────────────────┘
```

**Разбор:**
- Внешний контейнер: Box (чтобы время можно было поставить в правый нижний угол)
- Или: Column с Row для текста и времени

```kotlin
@Composable
fun MessageBubble(message: Message) {
    Column(
        modifier = Modifier
            .padding(horizontal = 16.dp, vertical = 4.dp)
            .background(
                color = if (message.isMine) MaterialTheme.colorScheme.primary
                        else MaterialTheme.colorScheme.surfaceVariant,
                shape = RoundedCornerShape(12.dp)  // Скруглённые углы
            )
            .padding(12.dp)  // Внутренний отступ (между фоном и текстом)
    ) {
        Text(
            text = message.text,
            color = if (message.isMine) Color.White else Color.Black
        )
        Text(
            text = message.time,
            style = MaterialTheme.typography.bodySmall,
            color = if (message.isMine) Color.White.copy(alpha = 0.7f)
                    else Color.Gray,
            modifier = Modifier.align(Alignment.End)  // Время прижато вправо
        )
    }
}
```

**Обрати внимание на порядок Modifier'ов:**
1. `.padding(horizontal = 16.dp, vertical = 4.dp)` — отступ от краёв экрана (margin)
2. `.background(...)` — фон пузыря
3. `.padding(12.dp)` — отступ внутри фона (padding) — чтобы текст не прилипал к краю пузыря

---

## Разбираем список контактов

```
┌──────────────────────────────────┐
│ [Аватар] Алиса           12:30  │
│          Привет! Как д...       │
├──────────────────────────────────┤
│ [Аватар] Борис           11:05  │
│          Да, конечно!           │
├──────────────────────────────────┤
│ [Аватар] Вика            Вчера  │
│          Когда встреча...       │
└──────────────────────────────────┘
```

Каждый элемент — **Row** (аватар слева, текст справа). Текст справа — **Column** (имя + последнее сообщение).

```kotlin
@Composable
fun ContactItem(contact: Contact) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .clickable { /* открыть чат */ }
            .padding(horizontal = 16.dp, vertical = 12.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        // Аватар
        Box(
            modifier = Modifier
                .size(48.dp)
                .clip(CircleShape)
                .background(MaterialTheme.colorScheme.primaryContainer)
        ) {
            Text(
                text = contact.name.first().toString(),
                modifier = Modifier.align(Alignment.Center),
                color = MaterialTheme.colorScheme.onPrimaryContainer
            )
        }

        Spacer(modifier = Modifier.width(12.dp))

        // Текстовая часть (имя + сообщение)
        Column(modifier = Modifier.weight(1f)) {
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween  // Имя слева, время справа
            ) {
                Text(
                    text = contact.name,
                    style = MaterialTheme.typography.titleSmall
                )
                Text(
                    text = contact.time,
                    style = MaterialTheme.typography.bodySmall,
                    color = Color.Gray
                )
            }
            Text(
                text = contact.lastMessage,
                style = MaterialTheme.typography.bodyMedium,
                color = Color.Gray,
                maxLines = 1,            // Только одна строка
                overflow = TextOverflow.Ellipsis  // ... в конце
            )
        }
    }
}
```

---

## Spacer: невидимый разделитель

`Spacer` — это пустой прямоугольник, который используется только для создания отступа. Аналогия: Spacer — как распорка в ящике, которая держит вещи на расстоянии.

```kotlin
Column {
    Text("Заголовок")
    Spacer(modifier = Modifier.height(16.dp))  // Отступ 16dp
    Text("Основной текст")
}

Row {
    Icon(Icons.Default.Person, null)
    Spacer(modifier = Modifier.width(8.dp))   // Отступ 8dp
    Text("Имя")
}
```

**Альтернатива:** Вместо Spacer можно использовать `Arrangement.spacedBy()` в Column/Row — это часто чище.

```kotlin
// Вместо Spacer между каждым элементом:
Column(verticalArrangement = Arrangement.spacedBy(16.dp)) {
    Text("Пункт 1")
    Text("Пункт 2")
    Text("Пункт 3")
}
```

---

## fillMaxSize, fillMaxWidth, wrapContent: размеры

| Modifier | Что делает | Аналогия |
|----------|-----------|----------|
| `fillMaxSize()` | Занять всё доступное пространство (и ширину, и высоту) | Расстелить ковёр на весь пол |
| `fillMaxWidth()` | Занять всю доступную ширину | Растянуть ковёр по ширине |
| `fillMaxHeight()` | Занять всю доступную высоту | Растянуть ковёр по длине |
| `wrapContentSize()` | Занять только столько, сколько нужно содержимому | Свернуть ковёр до размера подушки |
| `width(200.dp)` | Точная ширина | Ковёр ровно 2 метра |
| `heightIn(min = 48.dp)` | Минимальная высота | Ковёр не меньше 48dp |

```kotlin
// Кнопка на всю ширину с высотой 48dp
Button(
    onClick = { /* отправить */ },
    modifier = Modifier
        .fillMaxWidth()
        .height(48.dp)
) {
    Text("Отправить")
}

// Аватар фиксированного размера
Image(
    painter = painterResource("avatar.png"),
    contentDescription = "Аватар",
    modifier = Modifier.size(48.dp)  // 48 × 48 dp
)
```

---

## Weight: как делить пространство

`weight` работает как в Row, так и в Column. Он определяет, **какую долю свободного пространства** занимает элемент.

```kotlin
Row(modifier = Modifier.fillMaxWidth()) {
    Box(modifier = Modifier.weight(1f).height(40.dp).background(Color.Red))
    Box(modifier = Modifier.weight(2f).height(40.dp).background(Color.Blue))
    Box(modifier = Modifier.weight(1f).height(40.dp).background(Color.Green))
}
// Красный: 1/4 ширины | Синий: 2/4 (половина) | Зелёный: 1/4
```

**Практический пример** — поле ввода сообщения и кнопка:

```kotlin
Row(
    modifier = Modifier
        .fillMaxWidth()
        .padding(8.dp),
    verticalAlignment = Alignment.CenterVertically
) {
    // Поле ввода занимает почти всю ширину
    OutlinedTextField(
        value = text,
        onValueChange = { text = it },
        modifier = Modifier.weight(1f),  // ← Занимает свободное пространство
        placeholder = { Text("Сообщение...") }
    )

    Spacer(modifier = Modifier.width(8.dp))

    // Кнопка фиксированного размера
    IconButton(onClick = { /* отправить */ }) {
        Icon(Icons.Default.Send, "Отправить")
    }
}
```

---

## Alignment: выравнивание внутри контейнера

### В Column (вертикальный контейнер)

```kotlin
Column(
    horizontalAlignment = Alignment.Start          // ← Влево
    // horizontalAlignment = Alignment.CenterHorizontally  // ← По центру
    // horizontalAlignment = Alignment.End                // ← Вправо
) { ... }
```

### В Row (горизонтальный контейнер)

```kotlin
Row(
    verticalAlignment = Alignment.Top            // ← Вверху
    // verticalAlignment = Alignment.CenterVertically  // ← По центру
    // verticalAlignment = Alignment.Bottom            // ← Внизу
) { ... }
```

### В Box (наложение)

```kotlin
Box {
    Text("Внизу справа", modifier = Modifier.align(Alignment.BottomEnd))
    Text("По центру", modifier = Modifier.align(Alignment.Center))
    Text("Наверху слева", modifier = Modifier.align(Alignment.TopStart))
}
```

### Arrangement: распределение пространства

Помимо выравнивания, можно управлять тем, как элементы распределяются в контейнере:

```kotlin
Row(
    horizontalArrangement = Arrangement.SpaceBetween  // Первый влево, последний вправо, остальные равномерно
) {
    Text("Левый")
    Text("Правый")
}
// [Левый]                              [Правый]

Row(
    horizontalArrangement = Arrangement.SpaceEvenly  // Равные промежутки между всеми элементами
) {
    Text("А")
    Text("Б")
    Text("В")
}
// [А]      [Б]      [В]

Row(
    horizontalArrangement = Arrangement.Center  // Все элементы по центру
) {
    Text("А")
    Text("Б")
}
//        [А][Б]
```

---

## Scroll: прокрутка

### Вертикальная прокрутка (короткий контент)

```kotlin
Column(
    modifier = Modifier.verticalScroll(rememberScrollState())
) {
    // Много текста, но не список — просто прокрутка
    repeat(50) {
        Text("Строка $it")
    }
}
```

### LazyColumn: эффективный список (для чата, контактов)

`LazyColumn` — это как `RecyclerView` в Android или `ListView` в Flutter. Он создаёт только видимые элементы, а не все сразу. **Для списков и чатов используй LazyColumn, не обычный Column + scroll.**

```kotlin
LazyColumn(
    verticalArrangement = Arrangement.spacedBy(4.dp),
    reverseLayout = true  // Новые сообщения внизу = прокрутка снизу
) {
    items(messages) { message ->
        MessageBubble(message)
    }
}
```

---

## Сравнение с Flutter

Если ты знал Flutter, вот соответствие основных виджетов:

| Flutter | Compose | Что делает |
|---------|---------|-----------|
| `Column` | `Column` | Вертикальная стопка |
| `Row` | `Row` | Горизонтальный ряд |
| `Stack` | `Box` | Наложение |
| `Padding` | `Modifier.padding()` | Отступы |
| `SizedBox(width: 8)` | `Spacer(Modifier.width(8.dp))` | Пустой отступ |
| `Expanded` | `Modifier.weight(1f)` | Занять свободное пространство |
| `ListView.builder` | `LazyColumn` | Эффективный список |
| `Container` | `Modifier` (chain) | Обёртка с настройками |
| `Align` | `Modifier.align()` | Выравнивание в Box |
| `Center` | `Alignment.Center` | Центрирование |
| `Flexible` | `Modifier.weight()` | Гибкий размер |

### Ключевые отличия от Flutter

1. **Modifier вместо вложенных виджетов.** В Flutter: `Padding(padding: ..., child: Container(color: ..., child: Text(...)))`. В Compose: `Text(modifier = Modifier.padding(...).background(...))`. Меньше вложенности — код чище.

2. **Один padding вместо margin + padding.** В Flutter отдельные `padding` и `margin`. В Compose только `padding`, а порядок Modifier'ов определяет «внутри» или «снаружи».

3. **Arrangement.spacedBy вместо SizedBox между элементами.** В Flutter: `Column(children: [A, SizedBox(height: 8), B, SizedBox(height: 8), C])`. В Compose: `Column(verticalArrangement = Arrangement.spacedBy(8.dp)) { A(); B(); C() }`. Гораздо лаконичнее.

4. **Нет виджета-обёртки.** В Flutter почти любой стиль требует оборачивания в Container/SizedBox/Padding. В Compose всё цепляется через Modifier — нет лишней вложенности.

---

## Шпаргалка по Modifier

| Modifier | Что делает | Пример |
|----------|-----------|--------|
| `.padding(16.dp)` | Одинаковый отступ со всех сторон | `Modifier.padding(16.dp)` |
| `.padding(horizontal = 16.dp, vertical = 8.dp)` | Разные отступы по горизонтали/вертикали | |
| `.padding(start = 8.dp, top = 4.dp)` | Отступ с конкретных сторон | |
| `.fillMaxWidth()` | Занять всю ширину родителя | |
| `.fillMaxSize()` | Занять всё пространство | |
| `.width(200.dp)` / `.height(48.dp)` | Точный размер | |
| `.size(48.dp)` | Квадратный элемент 48×48 | |
| `.weight(1f)` | Занять долю свободного пространства | |
| `.background(Color.Blue)` | Фон (цвет) | |
| `.background(Color.Blue, RoundedCornerShape(12.dp))` | Фон со скруглёнными углами | |
| `.clip(CircleShape)` | Обрезать по форме круга | Для аватаров |
| `.clickable { }` | Сделать кликабельным | |
| `.align(Alignment.Center)` | Выровнять внутри Box | |
| `.offset(x = 4.dp, y = 2.dp)` | Сдвинуть относительно позиции | |
| `.border(1.dp, Color.Gray)` | Рамка | |
| `.alpha(0.5f)` | Полупрозрачность | |

---

## Чеклист: почему элемент не там, где надо?

Когда элемент не на месте, пройди по этому списку:

1. **Забыл `fillMaxWidth()`?** — По умолчанию элементы занимают только минимальное пространство. Если нужно на всю ширину — добавь.

2. **Забыл `verticalAlignment = Alignment.CenterVertically` в Row?** — Элементы выравниваются по верхнему краю. Если нужно по центру — укажи.

3. **Неправильный порядок Modifier'ов?** — `padding → background` даёт «margin», `background → padding` даёт «padding». Проверь порядок.

4. **Забыл `weight(1f)`?** — Если один элемент должен занять всё свободное пространство, а остальные — по содержимому, добавь `weight` на растягиваемый элемент.

5. **Нужен `Arrangement.spacedBy`?** — Если элементы слиплись, добавь расстояние между ними.

6. **Нужен Spacer?** — Иногда проще поставить явный `Spacer(Modifier.width(8.dp))`, чем настраивать Arrangement.

7. **Проверь Column vs Row.** — Если элементы идут вниз вместо вправо — возможно, ты используешь Column вместо Row.

8. **`Alignment` vs `Arrangement`.** — `Alignment` = где стоит один элемент. `Arrangement` = как распределены все элементы. Не путай.

---

➡️ [Словарь программиста ←](00-beginners-guide.md) | [Введение →](00-introduction.md) | [Основы Kotlin →](01-kotlin-basics.md)
