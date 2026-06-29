# Интеграция с API и сетевые запросы

> Сетевое взаимодействие — неотъемлемая часть большинства приложений. Этот раздел описывает работу с HTTP-клиентом Ktor 3.4.x, обработку JSON через `ContentNegotiation`, обработку ошибок и аутентификацию в кроссплатформенном контексте. Все версии и API актуальны на 30 июня 2026.

---

## 6.1. Ktor 3.4 — мультиплатформенный HTTP-клиент

[Ktor](https://ktor.io/) — фреймворк от JetBrains с двумя частями: серверной и клиентской. Для мобильной разработки нас интересует `ktor-client` — мультиплатформенный HTTP-клиент, работающий на:
- **Android** — через `OkHttp` или `Darwin` (можно использовать CIO)
- **iOS** — через `Darwin` engine (нативный `NSURLSession`)
- **Desktop (JVM)** — через `Java` engine или `CIO`
- **Web (Wasm/JS)** — через `Js` engine (Fetch API)

> **Важно:** `JsonFeature` устарел с Ktor 2.0 и удалён в Ktor 3.0. Вместо него используйте `ContentNegotiation` с `json()` функцией. Если видите в старых туториалах `install(JsonFeature)` — это устаревший код.

### Настройка

**`gradle/libs.versions.toml`:**
```toml
[versions]
ktor = "3.4.0"

[libraries]
ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor" }
ktor-client-okhttp = { module = "io.ktor:ktor-client-okhttp", version.ref = "ktor" }  # Android
ktor-client-darwin = { module = "io.ktor:ktor-client-darwin", version.ref = "ktor" }  # iOS
ktor-client-java = { module = "io.ktor:ktor-client-java", version.ref = "ktor" }      # Desktop
ktor-client-js = { module = "io.ktor:ktor-client-js", version.ref = "ktor" }          # Wasm/JS
ktor-content-negotiation = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktor" }
ktor-serialization-json = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktor" }
ktor-auth = { module = "io.ktor:ktor-client-auth", version.ref = "ktor" }
ktor-logging = { module = "io.ktor:ktor-client-logging", version.ref = "ktor" }
```

**`composeApp/build.gradle.kts`:**
```kotlin
kotlin {
    androidTarget()
    iosArm64()
    iosSimulatorArm64()
    jvm("desktop")
    wasmJs { browser() }

    sourceSets {
        commonMain.dependencies {
            implementation(libs.ktor.client.core)
            implementation(libs.ktor.content.negotiation)
            implementation(libs.ktor.serialization.json)
            implementation(libs.ktor.auth)
            implementation(libs.ktor.logging)
        }
        androidMain.dependencies {
            implementation(libs.ktor.client.okhttp)
        }
        iosMain.dependencies {
            implementation(libs.ktor.client.darwin)
        }
        val desktopMain by getting {
            dependencies {
                implementation(libs.ktor.client.java)
            }
        }
        val wasmJsMain by getting {
            dependencies {
                implementation(libs.ktor.client.js)
            }
        }
    }
}
```

### Создание клиента

```kotlin
import io.ktor.client.*
import io.ktor.client.engine.okhttp.*
import io.ktor.client.plugins.*
import io.ktor.client.plugins.auth.*
import io.ktor.client.plugins.auth.providers.*
import io.ktor.client.plugins.contentnegotiation.*
import io.ktor.client.plugins.logging.*
import io.ktor.serialization.kotlinx.json.*
import kotlinx.serialization.json.Json

// commonMain — единая фабрика клиента
fun createHttpClient(): HttpClient = HttpClient {
    install(ContentNegotiation) {
        json(Json {
            prettyPrint = false
            isLenient = true
            ignoreUnknownKeys = true  // важно для forward-compatibility
            explicitNulls = false     // не сериализовать null-поля
        })
    }

    install(Logging) {
        level = LogLevel.HEADERS
        logger = Logger.SIMPLE
    }

    install(HttpTimeout) {
        requestTimeoutMillis = 30_000
        connectTimeoutMillis = 10_000
        socketTimeoutMillis = 30_000
    }

    install(DefaultRequest) {
        url("https://api.example.com/")
        header("Accept", "application/json")
        header("X-Client-Version", "1.0.0")
    }

    expectSuccess = true  // бросать исключения на 4xx/5xx автоматически
}
```

> **`ignoreUnknownKeys = true` — критично:** когда бэкенд добавит новое поле, ваш клиент не упадёт с `UnknownFieldException`. Это правило forward-compatibility.

### Платформенные engine'ы

Engine выбирается автоматически по платформе, если вы добавили правильные зависимости. Можно также указать явно:

```kotlin
// androidMain
fun createAndroidClient(): HttpClient = HttpClient(OkHttp) {
    // конфигурация плагинов — общая через createHttpClient() или расширение
    engine {
        config {
            retryOnConnectionFailure(true)
        }
    }
}

// iosMain — Darwin engine использует NSURLSession
fun createIosClient(): HttpClient = HttpClient(Darwin) {
    engine {
        configureRequest {
            setAllowsCellularAccess(true)
        }
    }
}
```

---

## 6.2. Сериализация JSON: kotlinx.serialization

`kotlinx.serialization` — официальный фреймворк сериализации от JetBrains. В отличие от Gson или Jackson, он работает на всех платформах KMP и генерирует код сериализации **на этапе компиляции**. Это означает отсутствие рефлексии в runtime, что особенно важно для Kotlin/Native (iOS), где рефлексия ограничена.

### Подготовка моделей

```kotlin
import kotlinx.serialization.*

@Serializable
data class TaskDto(
    val id: String,
    val title: String,
    val completed: Boolean,
    val createdAt: String,  // ISO-8601 строка
    val assignee: String? = null,
    val tags: List<String> = emptyList()
)

@Serializable
data class CreateTaskRequest(
    val title: String,
    val tags: List<String> = emptyList()
)

@Serializable
data class TaskListResponse(
    val tasks: List<TaskDto>,
    val total: Int,
    val page: Int
)
```

### Выполнение запросов

```kotlin
class TaskApi(private val client: HttpClient) {

    // GET — список задач с пагинацией
    suspend fun getTasks(page: Int = 1): TaskListResponse {
        return client.get("tasks") {
            url {
                parameters.append("page", page.toString())
                parameters.append("size", "20")
            }
        }.body()
    }

    // GET — конкретная задача по ID
    suspend fun getTask(id: String): TaskDto {
        return client.get("tasks/$id").body()
    }

    // POST — создание задачи
    suspend fun createTask(request: CreateTaskRequest): TaskDto {
        return client.post("tasks") {
            contentType(ContentType.Application.Json)
            setBody(request)
        }.body()
    }

    // PATCH — обновление
    suspend fun updateTask(id: String, request: CreateTaskRequest): TaskDto {
        return client.patch("tasks/$id") {
            contentType(ContentType.Application.Json)
            setBody(request)
        }.body()
    }

    // DELETE
    suspend fun deleteTask(id: String) {
        client.delete("tasks/$id")
    }
}
```

> **Как это работает:** `ContentNegotiation`-плагин автоматически сериализует тело запроса (`setBody(request)`) в JSON и десериализует ответ (`.body<TaskDto>()`) в типобезопасный объект. Никаких ручных `Json.decodeFromString` — всё на этапе компиляции.

---

## 6.3. Обработка ошибок

Стратегия обработки ошибок должна быть **многоуровневой**:

1. **Сетевые исключения** — нет интернета, таймаут, DNS не резолвится.
2. **HTTP-ошибки 4xx** — клиентские ошибки (401 Unauthorized, 404 Not Found, 422 Validation).
3. **HTTP-ошибки 5xx** — серверные ошибки, временные.
4. **Ошибки десериализации** — сервер вернул непредвиденный формат.

### Иерархия исключений Ktor

```kotlin
import io.ktor.client.plugins.*

suspend fun safeFetch(api: TaskApi): Result<TaskDto> = runCatching {
    api.getTask("123")
}.recoverCatching { e ->
    when (e) {
        is ClientRequestException -> {
            // 4xx — ошибка от сервера с response
            val statusCode = e.response.status
            val errorBody = e.response.bodyAsText()
            throw when (statusCode) {
                HttpStatusCode.Unauthorized -> AuthException(errorBody)
                HttpStatusCode.NotFound -> NotFoundException("Task not found")
                HttpStatusCode.UnprocessableEntity -> ValidationException(errorBody)
                else -> ApiException("HTTP ${statusCode.value}: $errorBody")
            }
        }
        is ServerResponseException -> {
            // 5xx — повторить через какое-то время
            throw ServerUnavailableException()
        }
        is HttpRequestTimeoutException -> {
            throw TimeoutException()
        }
        is RedirectResponseException -> {
            // 3xx — обычно не должно происходить, expectSuccess=true бросает
            throw ApiException("Unexpected redirect")
        }
        is JsonConvertException -> {
            // Ошибка десериализации
            throw DataParseException(e.message)
        }
        else -> throw e  // сетевые ошибки, CancelledCoroutineException и т.д.
    }
}
```

### Ретраи для временных ошибок

Для временных ошибок (5xx, таймаут, нет сети) делайте retry с экспоненциальной задержкой:

```kotlin
suspend fun <T> retryWithBackoff(
    times: Int = 3,
    initialDelayMs: Long = 500,
    factor: Double = 2.0,
    block: suspend () -> T
): T {
    var currentDelay = initialDelayMs
    repeat(times - 1) { attempt ->
        try {
            return block()
        } catch (e: ServerResponseException) {
            // retry
        } catch (e: HttpRequestTimeoutException) {
            // retry
        } catch (e: IOException) {
            // нет сети — retry
        }
        delay(currentDelay)
        currentDelay = (currentDelay * factor).toLong()
    }
    return block()  // последняя попытка без catch
}
```

> **Альтернатива:** используйте готовый плагин `HttpRetry` (Ktor 3.1+) — он встроенный и проще в настройке:
> ```kotlin
> install(HttpRetry) {
>     retryOnException(maxRetries = 3)
>     retryOnServerErrors(maxRetries = 3)
>     exponentialDelay()
> }
> ```

---

## 6.4. Аутентификация

### Bearer Token

Плагин `Auth` автоматически добавляет токен к каждому запросу и обрабатывает обновление при истечении срока действия:

```kotlin
import io.ktor.client.plugins.auth.*
import io.ktor.client.plugins.auth.providers.*

// Хранилище токенов (обычно обёртка над multiplatform-settings)
interface TokenStorage {
    fun getAccessToken(): String?
    fun getRefreshToken(): String?
    fun saveTokens(access: String, refresh: String)
    fun clear()
}

fun createAuthenticatedClient(tokenStorage: TokenStorage): HttpClient = HttpClient {
    install(ContentNegotiation) { json() }

    install(Auth) {
        bearer {
            // 1. При каждом запросе — загружаем токен
            loadTokens {
                BearerTokens(
                    accessToken = tokenStorage.getAccessToken() ?: "",
                    refreshToken = tokenStorage.getRefreshToken() ?: ""
                )
            }

            // 2. При 401 — пытаемся обновить токен
            refreshTokens {
                val refreshToken = tokenStorage.getRefreshToken() ?: return@refreshTokens null
                val response = client.get("auth/refresh") {
                    header("X-Refresh-Token", refreshToken)
                }

                if (response.status == HttpStatusCode.Unauthorized) {
                    // Refresh токен истёк — выходим из аккаунта
                    tokenStorage.clear()
                    null
                } else {
                    val newTokens = response.body<TokenPair>()
                    tokenStorage.saveTokens(newTokens.accessToken, newTokens.refreshToken)
                    BearerTokens(newTokens.accessToken, newTokens.refreshToken)
                }
            }

            // 3. Отправляем токен на любые запросы, кроме /auth/*
            sendWithoutRequest { request ->
                !request.url.encodedPath.startsWith("/auth/")
            }
        }
    }
}
```

> **Как это работает:** когда сервер отвечает 401 на запрос с истёкшим access-токеном, Ktor автоматически вызывает `refreshTokens`, получает новую пару, сохраняет её и повторяет исходный запрос с новым токеном — прозрачно для вызывающего кода.

### Базовая аутентификация (Basic Auth)

```kotlin
install(Auth) {
    basic {
        credentials {
            BasicAuthCredentials(username = "user", password = "pass")
        }
    }
}
```

### API ключи (custom header)

Для API с ключом в заголовке не нужен плагин `Auth` — используйте `DefaultRequest`:

```kotlin
install(DefaultRequest) {
    header("X-API-Key", apiKey)
}
```

---

## 6.5. Загрузка файлов (multipart)

```kotlin
suspend fun uploadAvatar(
    api: TaskApi,
    imageBytes: ByteArray,
    fileName: String
): String {
    return api.client.submitFormWithBinaryData(
        url = "upload/avatar",
        formData = formData {
            append("description", "User avatar")
            append("file", imageBytes, Headers.build {
                append(HttpHeaders.ContentType, "image/jpeg")
                append(HttpHeaders.ContentDisposition, "filename=$fileName")
            })
        }
    ).body()
}
```

### Скачивание с прогрессом

```kotlin
suspend fun downloadFileWithProgress(
    url: String,
    onProgress: (sent: Long, total: Long?) -> Unit
): ByteArray {
    return client.get(url) {
        onDownload { sent, total ->
            onProgress(sent, total)
        }
    }.body()
}
```

---

## 6.6. WebSocket (real-time данные)

Ktor поддерживает WebSocket'ы для real-time приложений (чаты, обновления в реальном времени):

```kotlin
suspend fun connectToChat(api: TaskApi, roomId: String) {
    api.client.webSocket("ws://api.example.com/chat/$roomId") {
        // Отправка сообщений
        sendSerialized(ChatMessage(text = "Hello!"))

        // Приём
        while (true) {
            val message = receiveDeserialized<ChatMessage>()
            println("Received: ${message.text}")
        }
    }
}
```

> **Платформенные нюансы:** WebSocket работает на Android (OkHttp), iOS (Darwin), Desktop (Java), но на Web (Wasm/JS) использует браузерный WebSocket API. Поведение идентично.

---

## 6.7. Best practices

| Сценарий | Решение |
|----------|---------|
| **Сетевые ошибки** | Retry с экспоненциальной задержкой, fallback на кэш |
| **HTTP 401** | Автоматическое обновление токена через `refreshTokens` |
| **HTTP 422 (Validation)** | Десериализация тела ошибки в `ValidationErrorResponse`, показ ошибок пользователю |
| **HTTP 5xx** | Retry, затем fallback на кэш с индикатором «данные могут быть устаревшими» |
| **Таймаут** | Настроить в `HttpTimeout` (30s request, 10s connect) |
| **Forward-compatibility** | `ignoreUnknownKeys = true` в Json |
| **Логирование в debug** | `Logging` плагин с `LogLevel.HEADERS`, отключить в release |
| **Безопасность** | HTTPS-only, certificate pinning через OkHttp engine на Android |

---

⬅️ [← Работа с данными](05-data-storage.md) | [К содержанию](README.md) | [Локализация →](07-localization.md) ➡️
