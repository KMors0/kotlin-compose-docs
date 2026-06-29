# Руководство по миграции на Kotlin Multiplatform

> Пошаговая стратегия переноса существующего Android-проекта на KMP: от бизнес-логики до iOS UI.

---

⬅️ [← Анимации и Motion](29-animations-motion.md) | [К содержанию](README.md) | [Шпаргалка →](31-quick-reference.md) ➡️

---

## 30.1. Зачем мигрировать и когда это имеет смысл

| Область | Выигрыш | Пример экономии |
|--------|---------|------------------
| Бизнес-логика | Дублирование устраняется | ~40-60 % кода |
| Модели данных | Единые DTO и Domain-модели | 0 % дублирования |
| Сетевой слой | Один Ktor-клиент | ~50 % кода API |
| Тесты | Написываются один раз | ~70 % тестового кода |

**Миграция имеет смысл, если:** приложение на 100 % Kotlin; архитектура близка к Clean Architecture; планируется iOS-версия; команда готова инвестировать 2–6 месяцев.

**Миграция НЕ имеет смысла, если:** значительный объём Java; Activity/Fragment «толстые»; активно используются платформенные API (CameraX, BLE) без абстракций.

---

## 30.2. Фаза 1 — Вынос бизнес-логики в shared-модуль

Извлечение платформенно-независимого кода — можно сделать **до** подключения KMP-плагинов.

```
shared/src/
├── commonMain/kotlin/com/example/shared/
│   ├── domain/model/           # Task, User
│   ├── domain/repository/      # Интерфейсы репозиториев
│   ├── domain/usecase/         # GetTasksUseCase
│   └── data/dto/               # Сетевые модели
├── androidMain/kotlin/         # Android-реализации
└── iosMain/kotlin/             # iOS-реализации
```

```kotlin
// commonMain — Repository и Use Case
data class User(val id: String, val name: String, val email: String)
data class Result<out T>(val data: T?, val error: String?)

interface UserRepository {
    suspend fun getUsers(): Result<List<User>>
}

class GetUsersUseCase(private val repo: UserRepository) {
    suspend operator fun invoke() = repo.getUsers()
}
```

**Ключевое правило:** ни один файл в `commonMain` не импортирует `android.*`. Если такое происходит — абстрагируйте через `expect`/`actual`.

---

## 30.3. Фаза 2 — Подключение shared-модуля к Android-проекту

```kotlin
// shared/build.gradle.kts
plugins {
    alias(libs.plugins.kotlin.multiplatform)
    alias(libs.plugins.android.library)
}

kotlin {
    androidTarget()
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation(libs.kotlinx.coroutines.core)
                implementation(libs.ktor.client.core)
            }
        }
        val androidMain by getting {
            dependencies { implementation(libs.ktor.client.okhttp) }
        }
    }
}

android { namespace = "com.example.shared"; compileSdk = 35 }
```

```kotlin
// app/build.gradle.kts
dependencies { implementation(project(":shared")) }
```

```kotlin
// settings.gradle.kts
include(":app", ":shared")
```

---

## 30.4. Фаза 3 — Миграция UI на Compose

Если используется XML-разметка, миграция на Compose — обязательный шаг.

**Стратегия:** новые экраны — сразу на Compose; существующие — через `ComposeView`:

```kotlin
class ProfileFragment : Fragment() {
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View =
        ComposeView(requireContext()).apply {
            setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed)
            setContent { MaterialTheme { ProfileScreen(viewModel = viewModel()) } }
        }
}
```

ViewModel переносится в `commonMain`, если не зависит от Android API:

```kotlin
class TaskListViewModel(private val getTasks: GetTasksUseCase) : ViewModel() {
    private val _state = MutableStateFlow(TaskListState())
    val state: StateFlow<TaskListState> = _state.asStateFlow()
    fun loadTasks() { viewModelScope.launch { /* обновление _state */ } }
}
```

---

## 30.5. Фаза 4 — Добавление iOS-таргета

```kotlin
// shared/build.gradle.kts — добавление iOS
kotlin {
    androidTarget()
    listOf(iosX64(), iosArm64(), iosSimulatorArm64()).forEach {
        it.binaries.framework { baseName = "shared"; isStatic = true }
    }
    sourceSets {
        val commonMain by getting
        val iosMain by creating { dependsOn(commonMain) }
        val iosX64Main by getting { dependsOn(iosMain) }
        val iosArm64Main by getting { dependsOn(iosMain) }
        val iosSimulatorArm64Main by getting { dependsOn(iosMain) }
    }
    // CocoaPods (альтернатива SPM)
    cocoapods {
        summary = "Shared module"; ios.deploymentTarget = "14.0"
        podfile = project.file("../iosApp/Podfile")
    }
}
```

```ruby
# iosApp/Podfile
target 'iosApp' do
  use_frameworks!; platform :ios, '14.0'
  pod 'shared', :path => '../shared'
end
```

---

## 30.6. Фаза 5 — iOS UI

**Вариант A — SwiftUI + общий модуль** (рекомендуется для нативного ощущения):

```swift
import SwiftUI; import Shared
struct ContentView: View {
    @StateObject var vm = TaskListViewModelWrapper()
    var body: some View {
        List(vm.tasks, id: \.id) { task in Text(task.title).font(.headline) }
        .onAppear { vm.loadTasks() }
    }
}
```

**Вариант B — Compose Multiplatform** (общий UI): больше кода в `commonMain`, но меньше нативного ощущения.

---

## 30.7. Типичные проблемы и решения

**Android-specific API в общем коде → `expect`/`actual`:**

```kotlin
// commonMain
expect fun formatDate(timestamp: Long): String
// androidMain
actual fun formatDate(timestamp: Long) = SimpleDateFormat("dd.MM.yyyy", Locale.getDefault()).format(Date(timestamp))
// iosMain
actual fun formatDate(ts: Long): String = NSDateFormatter().apply { dateFormat = "dd.MM.yyyy" }
    .stringFromDate(NSDate(timeIntervalSince1970: ts / 1000.0))
```

**Зависимость от Context → интерфейсная абстракция:**

```kotlin
// commonMain — интерфейс
interface Analytics { fun logEvent(name: String, params: Map<String, String>) }
// androidMain
class AndroidAnalytics(private val ctx: Context) : Analytics { /* FirebaseAnalytics */ }
// iosMain
class IosAnalytics : Analytics { /* Firebase.Analytics */ }
```

**Lifecycle и CoroutineScope:** в `commonMain` передавайте `CoroutineScope` извне; Android использует `viewModelScope`, iOS — `MainScope()`.

---

## 30.8. Миграция зависимостей: AndroidX → KMP-аналоги

| AndroidX | KMP-аналог | |
|----------|-----------|-|
| Retrofit | Ktor Client | Полная замена |
| Room | SQLDelight | SQL-first подход |
| Gson / Moshi | kotlinx.serialization | Плагин компилятора |
| SharedPreferences | multiplatform-settings | Библиотека `russhwolf` |
| DataStore | KStore | Условная поддержка |
| Navigation Compose | Voyager / JetBrains Navigation | |
| Dagger / Hilt | Koin | Проще для KMP |
| Coil | Coil 3 / Kamel | Мультиплатформенные |
| ThreeTen BP | kotlinx-datetime | Стандарт де-факто |

---

## 30.9. Реалистичная оценка сроков

Для приложения среднего размера (~50K строк, ~30 экранов):

| Фаза | Срок | Что можно делать параллельно |
|------|------|-----------------------------|
| 1. Аудит + вынос логики | 2–3 нед. | Рефакторинг архитектуры |
| 2. Gradle + shared-модуль | 1 нед. | Настройка CI/CD |
| 3. Миграция на Compose | 3–6 нед. | Новые фичи сразу на Compose |
| 4. iOS-таргет | 1–2 нед. | Настройка Xcode |
| 5. iOS UI (SwiftUI) | 4–8 нед. | Параллельно с Android |
| 6. Стабилизация | 2–3 нед. | Общее тестирование |
| **Итого** | **3–6 мес.** | |

Для крупных проектов (> 200K строк) — умножайте на ×2.

---

## 30.10. Чек-лист готовности к миграции

- [ ] **Код на 100 % Kotlin** — Java-файлы отсутствуют или минимальны
- [ ] **Clean Architecture** — Domain не зависит от Android SDK
- [ ] **ViewModel без `Context`**, `Bundle`, `Intent` и других Android API
- [ ] **Репозитории имеют интерфейсы** в Domain-слое
- [ ] **Модели данных** — data class без `Parcelable`, `Bitmap` и т.п.
- [ ] **Сетевой слой** через интерфейсы (легко заменить Retrofit на Ktor)
- [ ] **DI** абстрагирован (Hilt-модули → Koin)
- [ ] **UI-тесты** покрывают критические сценарии
- [ ] **CI/CD** готов к сборке двух платформ
- [ ] **Команда владеет Swift** или есть бюджет на iOS-разработчиков
- [ ] **Stakeholders понимают**, что первые 2–4 недели — инвестиция без видимого функционала

---

⬅️ [← Анимации и Motion](29-animations-motion.md) | [К содержанию](README.md) | [Шпаргалка →](31-quick-reference.md) ➡️
