# Работа с данными и хранилищами

> В кроссплатформенных приложениях критически важно правильно организовать хранение и синхронизацию данных. В этой части рассматриваются основные подходы и инструменты для работы с данными в экосистеме Kotlin Multiplatform.

---

## 5.1. Кроссплатформенные ORM: SQLDelight

SQLDelight генерирует безопасный код для работы с SQLite.

**Шаги:**
1. Добавьте зависимости:
   ```kotlin
   implementation("com.squareup.sqldelight:runtime:$sqldelight_version")
   implementation("com.squareup.sqldelight:android-driver:$sqldelight_version") // для Android
   implementation("com.squareup.sqldelight:native-driver:$sqldelight_version") // для iOS
   ```
2. Создайте `.sq` файл с SQL-запросами.
3. Используйте сгенерированные классы для CRUD-операций.

**Пример (`Tasks.sq`):**
```sql
CREATE TABLE Task (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    completed BOOLEAN NOT NULL
);

INSERT INTO Task (id, title, completed) VALUES (?, ?, ?);
SELECT * FROM Task;
```

SQLDelight от Square — один из самых популярных инструментов для работы с базами данных в Kotlin Multiplatform. В отличие от Room (который работает только на Android), SQLDelight генерирует типобезопасный Kotlin-код из SQL-запросов, что позволяет обнаруживать ошибки на этапе компиляции, а не в runtime. Вы пишете SQL-запросы в `.sq`-файлах, а SQLDelight генерирует Kotlin-интерфейсы и реализации для каждой платформы.

---

## 5.2. Локальное хранение данных

- **Android:** `DataStore` (для примитивов и объектов), `SharedPreferences`.
- **iOS:** `UserDefaults`, `Keychain` (для чувствительных данных).
- **Web:** `localStorage`, `IndexedDB`.

**Пример DataStore:**
```kotlin
@Serializable
data class Settings(val theme: String = "light")

private val Context.dataStore by preferencesDataStore("settings")

val dataStore = context.dataStore

suspend fun saveTheme(theme: String) {
    dataStore.edit { preferences ->
        preferences[THEME_KEY] = theme
    }
}
```

Для кроссплатформенного подхода к локальному хранению простых данных (настройки, флаги) можно использовать `multiplatform-settings` — библиотеку, которая абстрагирует платформенные API (`SharedPreferences` на Android, `NSUserDefaults` на iOS, `localStorage` на Web) за единым интерфейсом. Это позволяет писать код работы с настройками один раз в общем модуле.

---

## 5.3. Синхронизация данных

Используйте паттерны:
- **Offline-first** — сначала работа с локальным хранилищем, затем синхронизация с сервером.
- **CQRS** — разделение команд и запросов.
- **Event Sourcing** — хранение истории изменений.

**Пример:** кэшируйте список задач локально, периодически обновляя его с сервера.

Паттерн **Offline-first** особенно важен для мобильных приложений, где интернет-соединение может быть нестабильным. При этом подходе приложение всегда читает данные из локального хранилища, обеспечивая мгновенный отклик UI. При появлении сети данные синхронизируются с сервером в фоновом режиме. Для разрешения конфликтов используйте метки времени или версионность.

**Пример реализации Offline-first:**
```kotlin
class TasksRepository(
    private val localDataSource: TaskLocalDataSource,
    private val remoteDataSource: TaskRemoteDataSource
) {
    fun getTasks(): Flow<List<Task>> = localDataSource.getAll()

    suspend fun sync() {
        val remoteTasks = remoteDataSource.fetchTasks()
        localDataSource.replaceAll(remoteTasks)
    }

    suspend fun addTask(task: Task) {
        localDataSource.insert(task)
        try {
            remoteDataSource.createTask(task)
        } catch (e: Exception) {
            // Помечаем для последующей синхронизации
        }
    }
}
```

---

⬅️ [← Практический пример](04-practical-example.md) | [К содержанию](README.md) | [Сетевые запросы →](06-networking-api.md) ➡️
