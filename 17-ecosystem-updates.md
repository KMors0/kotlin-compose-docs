# Обновления экосистемы

> Экосистема Kotlin и Compose Multiplatform в 2026 году развивается рекордными темпами. Этот раздел фиксирует актуальное состояние стека на **30 июня 2026 года** — версии, ключевые релизы, новые API и направление дорожной карты. Если вы читаете этот раздел спустя несколько месяцев, обязательно сверяйтесь с официальными источниками (kotlinlang.org, blog.jetbrains.com), так как релизы выходят каждые 6–8 недель.

---

## 17.1. Версионный снимок стека (на 30 июня 2026)

| Технология | Stable | RC / Beta | Ключевое в релизе |
|------------|--------|-----------|-------------------|
| **Kotlin** | 2.4.0 (также патч 2.3.21 от 23 апреля 2026) | 2.4.10-RC, 2.4.20-Beta1 | Swift packages как зависимости, обновлённый Swift export, новые языковые фичи (в 2.4.20-Beta1 — coroutine stack trace recovery в stdlib) |
| **Compose Multiplatform** | 1.11.0 | — | Unified `@Preview` (с 1.10), поддержка Navigation 3 на non-Android target, refreshed UI testing, Skia M138 |
| **AndroidX Compose** | 1.11.2 (май 2026) | 1.12.0-alpha | Стабилизация новых M3 Expressive API |
| **compose-material3** | 1.4.x (M3 Expressive) | — | LoadingIndicator, SplitButton, FAB Menu, Motion physics |
| **kotlinx.coroutines** | 1.11.0 (9 мая 2026) | — | Terminal Flow-операторы (`collectToMap` и др.) |
| **Ktor** | 3.4.0+ | — | Zstd compression, SSE improvements, Ktor client for Kotlin/Wasm |
| **SQLDelight** | 2.1+ | — | Поддержка New Memory Manager, адаптеры для KMP |
| **Koin** | 4.0+ | — | Multiplatform-native DI, улучшенная интеграция с Compose |
| **Decompose** | 3.x | — | Стабильные lifecycle/state API для KMP |
| **Voyager** | 1.1+ | — | Альтернативная навигация для Compose Multiplatform |
| **skiko (Skia для Kotlin)** | M138 | — | Используется в CMP 1.11 для рендеринга на iOS/Desktop/Web |

> **Как читать таблицу:** Stable — версия, рекомендуемая для production. RC (Release Candidate) — финальная стабилизация, можно пробовать в проектах с готовностью к минорным правкам. Beta — ранняя версия, подходит для экспериментов.

---

## 17.2. Дорожная карта Compose Multiplatform

JetBrains официально опубликовала дорожную карту Compose Multiplatform на 2026 год. Ключевые направления:

### Что уже стабильно (можно использовать в production)

- **iOS-бэкенд** — Stable с версии 1.8.0 (май 2025). Рендеринг через Skia поверх Metal/Core Animation, поддержка VoiceOver, AssistiveTouch, Full Keyboard Access, нативные 120 Гц дисплеи.
- **Desktop (JVM)** — Stable. Skia через OpenGL, поддержка JDK 11+, нативное оконирование через Compose for Desktop.
- **Web (Kotlin/Wasm)** — Beta → Stable-готовность в 1.11.0. Wasm-бэкенд пришёл на смену Kotlin/JS-таргету; быстрее, лучше интеграция с браузером, доступ к `WebElementView()` для встраивания HTML-элементов.
- **Unified `@Preview`** (с 1.10.0) — единая аннотация `@Preview` для всех платформ (Android, iOS, Desktop, Wasm), без необходимости держать Android Studio только для preview.
- **Navigation 3** на non-Android targets — навигационная библиотека AndroidX Navigation 3 работает на iOS/Desktop/Web через Compose Multiplatform.

### Что в активной разработке (Beta / EAP)

- **Compose Multiplatform 1.12.0** (ожидается осенью 2026) — фокус на улучшении производительности Web-бэкенда, расширенная поддержка accessibility API на iOS.
- **Skia M138+** — обновление графического движка, улучшения в производительности рендеринга текста и сложных путей.
- **K2 compiler plugins** — новое поколение плагинов компилятора (Compose Compiler, serialization и др.), работает на базе K2 с Kotlin 2.x.

### Ключевые вехи 2026 года

- **Май 2026** — Compose Multiplatform 1.11.0 с обновлённым подходом к UI testing. Теперь можно писать тесты, работающие на всех платформах без модификаций.
- **Март 2026** — Kotlin 2.3.0 с языковыми улучшениями и стабилизацией K2.
- **Июнь 2026** — Kotlin 2.4.0 stable с поддержкой Swift packages как зависимостей — это упрощает интеграцию с нативными iOS-библиотеками.
- **23 июня 2026** — Kotlin 2.4.20-Beta1 (EAP), первая версия с поддержкой coroutine stack trace recovery в standard library.

---

## 17.3. Сравнение Material Design 3 vs M3 Expressive

Material 3 Expressive — расширение Material Design 3, анонсированное Google в 2025 году и стабилизированное в `compose-material3` 1.4.x в 2026 году. Это не новая версия, а надстройка: вы можете использовать классические M3-компоненты и Expressive-компоненты в одном приложении.

| Аспект | MD3 (классический) | M3 Expressive |
|--------|--------------------|--------------------|
| Цветовая система | Динамические цвета (Dynamic Color) на Android 12+ | Те же + новые «вибрантные» палитры с более насыщенными оттенками |
| Типографика | 15 ролей (display/headline/title/body/label × S/M/L) | Те же 15 ролей + расширенная поддержка шрифтовых вариаций и весов |
| Формы (Shapes) | 5 размеров (extraSmall → extraLarge), скруглённые | Расширенная библиотека: асимметричные формы, формы «react-to-context», экстремальные скругления |
| Motion | tween/spring-анимации, основные easing-функции | Motion physics — встроенные физически корректные motion-schemes, container transform, shared axis |
| Кнопки | Button, FilledTonalButton, OutlinedButton, TextButton, ElevatedButton, IconButton, FloatingActionButton, ExtendedFloatingActionButton | Те же + **SplitButton** (композитная кнопка с основным и вторичным действиями), **FAB Menu** (раскрывающееся меню из FAB, замена speed dial) |
| Индикаторы загрузки | CircularProgressIndicator, LinearProgressIndicator | Те же + **LoadingIndicator** и **ContainedLoadingIndicator** — новые компоненты с физикой движения и attention-holding поведением |
| TopAppBar | `TopAppBar`, `MediumTopAppBar`, `LargeTopAppBar` | Те же, с улучшенным collapse-поведением и интеграцией с motion schemes |

### Пример: новые компоненты M3 Expressive в коде

```kotlin
import androidx.compose.material3.*
import androidx.compose.material3.expressive.*

@Composable
fun ExpressiveExample() {
    Column {
        // Новый LoadingIndicator с motion physics
        LoadingIndicator(
            modifier = Modifier.padding(16.dp)
        )

        // ContainedLoadingIndicator — версия внутри контейнера
        ContainedLoadingIndicator(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp)
        )

        // SplitButton — основное действие + вторичное
        SplitButton(
            mainButton = {
                Button(onClick = { /* основное действие */ }) {
                    Text("Сохранить")
                }
            },
            secondaryButton = {
                SplitButtonDefaults.SecondaryButton(
                    onClick = { /* меню опций */ },
                    content = { Icon(Icons.Default.MoreVert, "Опции") }
                )
            }
        )

        // FAB Menu — раскрывающееся меню
        FloatingActionButtonMenu(
            expandableFab = {
                ExpandableFloatingActionButton(
                    onClick = { /* toggle */ },
                    expanded = false
                )
            },
            content = {
                FloatingActionButtonMenuItem(
                    onClick = { /* action 1 */ },
                    icon = { Icon(Icons.Default.Add, "Создать") },
                    text = { Text("Создать") }
                )
                FloatingActionButtonMenuItem(
                    onClick = { /* action 2 */ },
                    icon = { Icon(Icons.Default.Share, "Поделиться") },
                    text = { Text("Поделиться") }
                )
            }
        )
    }
}
```

> **Совет по миграции:** новые M3 Expressive компоненты помечены аннотацией `@ExperimentalMaterial3Api`. Перед использованием в production добавьте `@OptIn(ExperimentalMaterial3Api::class)` или включите оптику через `-opt-in=androidx.compose.material3.ExperimentalMaterial3Api` в `build.gradle.kts`.

---

## 17.4. Тренды и сообщества

### Сообщество

- **JetBrains Compose Slack / Discord** — основное место для оперативных вопросов; JetBrains-инженеры отвечают в канале `#compose-multiplatform`. Slack: `slack.kotlinlang.org`.
- **GitHub** — репозиторий [JetBrains/compose-multiplatform](https://github.com/JetBrains/compose-multiplatform) с issues, discussions и samples. Тег `compose-multiplatform` на GitHub даёт сотни open-source библиотек.
- **KotlinConf 2026** — ежегодная конференция JetBrains, в 2026 прошла в Копенгагене. Все доклады выкладываются на YouTube-канале JetBrainsTV.
- **Kotlin/Everywhere** — серия митапов по всему миру, организуемых при поддержке Google и JetBrains.

### Тренды 2026 года

1. **Сдвиг от MVVM к MVI+UDF в KMP-проектах** — Compose-нативная унификация через `Snapshot.StateFlow` и `derivedStateOf` делает MVI-цикл дешевле и предсказуемее.
2. **Замена KSP1 на KSP2** — начиная с Kotlin 2.4, KSP2 — рекомендуемый путь для annotation processors, существенно быстрее.
3. **Swift export из Kotlin** — с Kotlin 2.4 генерация Swift-фреймворков из Kotlin-модулей стала стабильной, что упрощает интеграцию KMP-модулей в iOS-проекты на чистом Swift (без Objective-C bridging).
4. **Navigation 3 на non-Android** — стандартная AndroidX-навигация работает на всех платформах, что снижает необходимость в Decompose/Voyager для новых проектов.
5. **Wasm как основной web-таргет** — Kotlin/JS постепенно выводится из фокуса, Kotlin/Wasm (Wasm GC) — рекомендуемый путь для Compose for Web.

### Рекомендуемые ресурсы для отслеживания обновлений

- [Compose Multiplatform GitHub](https://github.com/JetBrains/compose-multiplatform) — releases и changelogs.
- [JetBrains Blog](https://blog.jetbrains.com/kotlin/) — официальные анонсы.
- [Kotlin Blog (kotlinlang.org)](https://kotlinlang.org/docs/whatsnew.html) — What's new в каждой версии.
- [AndroidX Release Notes](https://developer.android.com/jetpack/androidx/versions) — стабильные и альфа-релизы AndroidX Compose.
- [compose-material3 reference](https://developer.android.com/jetpack/androidx/releases/compose-material3) — обновления Material 3 Expressive компонентов.
- [#compose на Kotlin Slack](https://slack.kotlinlang.org/) — живое обсуждение.
- [KotlinConf YouTube](https://www.youtube.com/@JetBrainsTV) — записи докладов.

---

## 17.5. Чеклист обновления существующего проекта

Если у вас проект на устаревших версиях (Kotlin 2.1.x, CMP 1.7.x), порядок обновления:

1. **Сначала Kotlin** — поднимите Kotlin до 2.4.0 в `libs.versions.toml`. Обновите KSP, Compose Compiler Plugin, serialization plugin до версий, совместимых с Kotlin 2.4.
2. **Потом Compose Multiplatform** — поднимите `org.jetbrains.compose` до 1.11.0. Проверьте `compose.kotlin.native.mode` — для production используйте `native` (не `legacy`).
3. **Библиотеки** — обновите kotlinx-coroutines до 1.11.0, Ktor до 3.4.0, SQLDelight до 2.1+.
4. **Проверьте warnings** — Kotlin 2.4 добавил новые deprecation warnings для устаревших API (например, `kotlin.native.concurrent.freeze` полностью убран в New MM).
5. **Прогон тестов на всех платформах** — `./gradlew allTests` обязательно. Особенно iOS — после обновления CMP могут вскрыться проблемы с новыми lifecycle API.
6. **Профилирование** — после обновления обязательно прогоните на реальном iOS-устройстве, особенно если обновили Skia (через skiko). Производительность может как улучшиться (новая Skia), так и ухудшиться (регрессии в рендеринге).

> **Важно:** Не обновляйте Kotlin и CMP одновременно в одном PR. Если что-то сломается, будет сложно понять, кто виноват. Сначала Kotlin → проверка → потом CMP → проверка.

---

⬅️ [← Примеры приложений](16-real-examples.md) | [К содержанию](README.md) | [Стиль кода →](18-code-style.md) ➡️
