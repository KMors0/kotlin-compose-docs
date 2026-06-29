# Работа с данными и хранилищами

> В кроссплатформенных приложениях критически важно правильно организовать хранение и синхронизацию данных. Этот раздел описывает три ключевых инструмента: SQLDelight (типобезопасный SQL), multiplatform-settings (key-value хранилище) и паттерны offline-first синхронизации. Все версии и API актуальны на 30 июня 2026 (SQLDelight 2.1+, multiplatform-settings 1.2+, DataStore 1.1+).

---

## 5.1. SQLDelight — типобезопасная работа с SQLite

[SQLDelight](https://github.com/cashapp/sqldelight) от Cash App (ранее Square) генерирует типобезопасный Kotlin-код из SQL-запросов. В отличие от Room (только Android), SQLDelight работает на всех платформах KMP. Вы пишете SQL в `.sq`-файлах, библиотека генерирует Kotlin-интерфейсы и реализации для каждой платформы. Ошибки в запросах обнаруживаются на этапе компиляции, а не в runtime.

> **Важно:** groupId библиотеки изменился с `com.squareup.sqldelight` (1.x) на `app.cash.sqldelight` (2.x). Если видите старый groupId в чужих туториалах — это устаревший код.

### Настройка

**`gradle/libs.versions.toml`:**
```toml
[versions]
sqldelight = "2.1.0"

[libraries]
sqldelight-runtime = { module = "app.cash.sqldelight:runtime", version.ref = "sqldelight" }
sqldelight-android = { module = "app.cash.sqldelight:android-driver", version.ref = "sqldelight" }
sqldelight-native = { module = "app.cash.sqldelight:native-driver", version.ref = "sqldelight" }
sqldelight-sqlite = { module = "app.cash.sqldelight:sqlite-driver", version.ref = "sqldelight" }  # для Desktop
sqldelight-coroutines = { module = "app.cash.sqldelight:coroutines-extensions", version.ref = "sqldelight" }

[plugins]
sqldelight = { id = "app.cash.sqldelight", version.ref = "sqldelight" }
```

**`composeApp/build.gradle.kts`:**
```kotlin
plugins {
    alias(libs.plugins.sqldelight)
}

kotlin {
    androidTarget()
    iosArm64()
    iosSimulatorArm64()
    jvm("desktop")

    sourceSets {
        commonMain.dependencies {
            implementation(libs.sqldelight.runtime)
            implementation(libs.sqldelight.coroutines)  // Flow API для SQLDelight
        }
        androidMain.dependencies {
            implementation(libs.sqldelight.android)
        }
        iosMain.dependencies {
            implementation(libs.sqldelight.native)
        }
        val desktopMain by getting {
            dependencies {
                implementation(libs.sqldelight.sqlite)
            }
        }
    }
}

sqldelight {
    databases {
        create("AppDatabase") {
            packageName.set("com.example.app.db")
            // src/commonMain/sqldelight/com/example/app/db/
        }
    }
}
```

### Создание схемы

Создайте файл `composeApp/src/commonMain/sqldelight/com/example/app/db/Tasks.sq`:

```sql
CREATE TABLE Task (
    id TEXT NOT NULL PRIMARY KEY,
    title TEXT NOT NULL,
    completed INTEGER AS Boolean DEFAULT 0,
    created_at INTEGER AS Instant
);

-- SQLDelight генерирует функции Kotlin по имени запроса
insertTask:
INSERT INTO Task(id, title, completed, created_at)
VALUES (?, ?, ?, ?);

selectAll:
SELECT * FROM Task
ORDER BY created_at DESC;

selectById:
SELECT * FROM Task WHERE id = ?;

selectCompleted:
SELECT * FROM Task WHERE completed = 1;

updateCompleted:
UPDATE Task SET completed = ? WHERE id = ?;

deleteById:
DELETE FROM Task WHERE id = ?;

countByCompleted:
SELECT COUNT(*) FROM Task WHERE completed = ?;
```

**Что генерирует SQLDelight:**
- `Task` — data class со всеми полями (с маппингом типов: `INTEGER AS Boolean` → `Boolean`, `INTEGER AS Instant` → `Instant` из kotlinx-datetime).
- `AppDatabase` — точка доступа к запросам: `appDatabase.taskQueries.insertTask(...)`, `appDatabase.taskQueries.selectAll()`.
- Типизированные функции — параметры проверяются на этапе компиляции.

### Платформенные драйверы

```kotlin
// commonMain
expect class DatabaseDriverFactory {
    fun createDriver(): SqlDriver
}

// androidMain
actual class DatabaseDriverFactory(private val context: Context) {
    actual fun createDriver(): SqlDriver =
        AndroidSqliteDriver(AppDatabase.Schema, context, "app.db")
}

// iosMain
actual class DatabaseDriverFactory {
    actual fun createDriver(): SqlDriver =
        NativeSqliteDriver(AppDatabase.Schema, "app.db")
}

// desktopMain
actual class DatabaseDriverFactory {
    actual fun createDriver(): SqlDriver =
        JdbcSqliteDriver(JdbcSqliteDriver.IN_MEMORY)  // или путь к файлу
}
```

### Использование в коде

```kotlin
class TaskRepository(private val database: AppDatabase) {

    private val queries get() = database.taskQueries

    // Синхронный запрос
    fun getTaskById(id: String): Task? =
        queries.selectById(id).executeAsOneOrNull()

    // Flow-наблюдение за изменениями (требует sqldelight-coroutines-extensions)
    fun observeAllTasks(): Flow<List<Task>> =
        queries.selectAll()
            .asFlow()
            .asList()

    fun observeCompleted(): Flow<List<Task>> =
        queries.selectCompleted()
            .asFlow()
            .asList()

    // Мутация (suspend, чтобы вызывать из корутин)
    suspend fun addTask(title: String): Task {
        val task = Task(
            id = UUID.randomUUID().toString(),
            title = title,
            completed = false,
            createdAt = Clock.System.now()
        )
        queries.transaction {
            queries.insertTask(
                id = task.id,
                title = task.title,
                completed = task.completed,
                created_at = task.createdAt
            )
        }
        return task
    }

    suspend fun toggleTask(id: String) {
        queries.transaction {
            val task = queries.selectById(id).executeAsOneOrNull() ?: return@transaction
            queries.updateCompleted(completed = !task.completed, id = id)
        }
    }

    suspend fun deleteTask(id: String) {
        queries.deleteById(id)
    }
}
```

> **Tip:** `transaction { }` — атомарная операция. Все запросы внутри выполняются в одной транзакции, при ошибке откатывается целиком. Используйте для операций, которые должны быть согласованными (например, «удалить задачу и связанные с ней напоминания»).

### Миграции

При изменении схемы создавайте миграции:

```kotlin
// composeApp/src/commonMain/sqldelight/com/example/app/db/2.sqm
ALTER TABLE Task ADD COLUMN priority INTEGER DEFAULT 0;
```

Каждая миграция — это файл `N.sqm`, где N — номер версии. SQLDelight проверяет миграции на этапе компиляции, чтобы гарантировать, что они приводят к ожидаемой схеме.

```kotlin
val driver = AndroidSqliteDriver(
    schema = AppDatabase.Schema,
    context = context,
    name = "app.db",
    callback = object : SqlDriver.Callback {
        override fun onConfigure(driver: SqlDriver) {
            // Выполняется при создании БД
        }
    }
)
```

---

## 5.2. Key-value хранилище: multiplatform-settings

Для простых данных (настройки, флаги, токены) не нужна SQLite — используйте [multiplatform-settings](https://github.com/russhwolf/multiplatform-settings). Эта библиотека абстрагирует платформенные API (`SharedPreferences`/`DataStore` на Android, `NSUserDefaults` на iOS, `localStorage` на Web) за единым интерфейсом.

**Зависимости:**
```toml
[versions]
multiplatform-settings = "1.2.0"

[libraries]
settings-core = { module = "com.russhwolf:multiplatform-settings", version.ref = "multiplatform-settings" }
settings-no-arg = { module = "com.russhwolf:multiplatform-settings-no-arg", version.ref = "multiplatform-settings" }
settings-coroutines = { module = "com.russhwolf:multiplatform-settings-coroutines", version.ref = "multiplatform-settings" }
settings-datastore = { module = "com.russhwolf:multiplatform-settings-datastore", version.ref = "multiplatform-settings" }  # Android-only
```

**Настройка платформенных реализаций:**
```kotlin
// commonMain
expect class SettingsFactory {
    fun create(): Settings
}

// androidMain — используется SharedPreferences (через no-arg delegate)
actual class SettingsFactory(private val context: Context) {
    actual fun create(): Settings = SharedPreferencesSettings(
        context.getSharedPreferences("app_prefs", Context.MODE_PRIVATE)
    )
}

// iosMain
actual class SettingsFactory {
    actual fun create(): Settings = NSUserDefaultsSettings()
}

// wasmJsMain
actual class SettingsFactory {
    actual fun create(): Settings = localStorageSettings()
}
```

**Использование:**
```kotlin
class SettingsRepository(private val settings: Settings) {

    var theme: String
        get() = settings.getStringOrNull("theme") ?: "system"
        set(value) = settings.putString("theme", value)

    var notificationsEnabled: Boolean
        get() = settings.getBooleanOrNull("notifications") ?: true
        set(value) = settings.putBoolean("notifications", value)

    var userId: String?
        get() = settings.getStringOrNull("user_id")
        set(value) = settings.putString("user_id", value)

    // Flow для наблюдения за изменениями (требует settings-coroutines)
    fun observeTheme(): Flow<String?> =
        settings.toFlow().getStringOrNullFlow("theme")
}
```

> **Когда выбирать multiplatform-settings vs SQLDelight:** multiplatform-settings — для пар «ключ-значение», обычно меньше 50 ключей, без сложных запросов. SQLDelight — для структурированных данных со связями, индексами, JOIN'ами.

### DataStore на Android (для случаев когда нужен typed storage)

Если в проекте уже есть Android-only DataStore и вы не хотите мигрировать на SQLDelight/multiplatform-settings — можно использовать [multiplatform-settings-datastore](https://github.com/russhwolf/multiplatform-settings) модуль, который обёртывает Jetpack DataStore под KMP-интерфейс `Settings`. На других платформах под капотом будет localStorage/NSUserDefaults.

```kotlin
// androidMain
actual class SettingsFactory(private val context: Context) {
    actual fun create(): Settings = DataStoreSettings(
        delegate = context.dataStore
    )
}

private val Context.dataStore by preferencesDataStore("app_prefs")
```

---

## 5.3. Синхронизация: паттерн Offline-First

**Offline-first** — паттерн, при котором приложение всегда читает данные из локального хранилища (мгновенный отклик UI), а при появлении сети синхронизирует с сервером в фоне. Это критически важно для мобильных приложений, где интернет-соединение может быть нестабильным.

### Архитектура репозитория

```kotlin
class TasksRepository(
    private val localDataSource: TaskLocalDataSource,    // SQLDelight
    private val remoteDataSource: TaskRemoteDataSource,  // Ktor
    private val syncQueue: SyncQueue                      // очередь неотправленных операций
) {
    // UI всегда читает из локальной БД — мгновенный отклик
    fun observeTasks(): Flow<List<Task>> =
        localDataSource.observeAll()

    // Создание задачи — пишем локально + в очередь синхронизации
    suspend fun addTask(title: String): Task {
        val task = Task(
            id = UUID.randomUUID().toString(),
            title = title,
            completed = false,
            createdAt = Clock.System.now(),
            syncState = SyncState.PENDING_CREATE  // помечаем как несинхронизированную
        )
        localDataSource.insert(task)
        syncQueue.enqueue(SyncOperation.Create(task))

        // Если есть сеть — сразу пытаемся отправить
        if (isOnline()) {
            trySync()
        }
        return task
    }

    // Фоновая синхронизация
    suspend fun trySync() {
        val pending = syncQueue.getAll()
        for (operation in pending) {
            try {
                when (operation) {
                    is SyncOperation.Create -> {
                        val remoteId = remoteDataSource.createTask(operation.task)
                        localDataSource.updateRemoteId(operation.task.id, remoteId)
                    }
                    is SyncOperation.Update -> remoteDataSource.updateTask(operation.task)
                    is SyncOperation.Delete -> remoteDataSource.deleteTask(operation.task.id)
                }
                syncQueue.markDone(operation.id)
            } catch (e: Exception) {
                // Логируем, оставляем в очереди для следующей попытки
                continue
            }
        }
    }
}
```

### Разрешение конфликтов

Когда данные могут измениться одновременно локально и на сервере, нужна стратегия разрешения конфликтов:

| Стратегия | Описание | Когда использовать |
|-----------|----------|--------------------|
| **Last-Write-Wins (LWW)** | Побеждает изменение с более поздним `updatedAt` | Простые данные, где потеря изменения не критична |
| **Versioned ( optimistic concurrency)** | Сервер отвергает обновление, если версия не совпадает | Данные где важна консистентность |
| **Merge** | Сервер сливает изменения полей | Сложные объекты (например, документы) |
| **Manual** | Показываем пользователю конфликт, он выбирает | Чаты, документы с совместным редактированием |

```kotlin
// Пример LWW-конфликта
suspend fun syncTask(localTask: Task) {
    try {
        remoteDataSource.updateTask(localTask)
    } catch (e: ConflictException) {
        val serverTask = remoteDataSource.getTask(localTask.id)
        if (serverTask.updatedAt > localTask.updatedAt) {
            // Серверская версия новее — перезаписываем локальную
            localDataSource.insert(serverTask.copy(syncState = SyncState.SYNCED))
        } else {
            // Локальная версия новее — повторяем отправку с forceUpdate
            remoteDataSource.forceUpdateTask(localTask)
        }
    }
}
```

> [!INFO]
> **`.copy(...)`** — метод, автоматически генерируемый для `data class` в Kotlin. Создаёт новую копию объекта с изменёнными полями. `serverTask.copy(syncState = SyncState.SYNCED)` создаёт новый `Task` со всеми полями как у `serverTask`, но с `syncState = SYNCED`. Оригинальный `serverTask` не изменяется (immutable). Это идиоматичный способ «обновить» одно поле в immutable-объекте.
>
> **`is SyncOperation.Update`** (в примере `when (operation)`) — smart cast: после `is SyncOperation.Update` компилятор автоматически приводит `operation` к типу `SyncOperation.Update`, и можно обращаться к `operation.task` без явного приведения.

### Когда выбирать Offline-First

- **Нужно:** мессенджеры, todo-приложения, field-приложения (для курьеров, ремонтников), приложения с пользовательским контентом (notes, фото).
- **Не нужно:** приложения с преимущественно серверным контентом (новости, погода), где офлайн не даёт UX-выгод; приложения с чувствительными данными, где пользователь должен видеть только свежие данные (банк, биржа).

---

⬅️ [← Практический пример](04-practical-example.md) | [К содержанию](README.md) | [Сетевые запросы →](06-networking-api.md) ➡️
