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

> [!TIP]
> **`runCatching { ... }`** — встроенная функция Kotlin, оборачивает результат в `Result<T>`. Если блок выполнился успешно — `Result.success(value)`. Если бросил исключение — `Result.failure(exception)`. Эквивалент `try { Result.success(block()) } catch (e) { Result.failure(e) }`, но без boilerplate.
>
> **`recoverCatching { e -> ... }`** — метод `Result`, позволяет заменить ошибку на новое значение (или пробросить другую ошибку). Лямбда получает исключение `e` и может либо вернуть значение, либо бросить новое исключение. В примере мы перехватываем исключения Ktor и пробрасываем их как более конкретные доменные исключения.
>
> **`is` в `when (e) { is ClientRequestException -> ... }`** — проверка типа с smart cast. После `is ClientRequestException` компилятор автоматически приводит `e` к `ClientRequestException`, и можно обращаться к `e.response` без явного приведения.
>
> **`when` как expression** — когда `when` используется как выражение (присваивается переменной или возвращается), Kotlin требует, чтобы все ветки были покрыты (или была ветка `else`). Внутри `throw when (...)` результат `when` — это исключение, которое сразу бросается.

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

## 6.7. HTTPS, TLS и QUIC/HTTP/3

Большинство сетевых запросов сегодня идут через HTTPS — это HTTP поверх TLS. Понимание, как TLS работает в KMP-приложениях, критично для безопасности и производительности.

### TLS — что происходит под капотом

При `https://`-запросе происходит рукопожатие (TLS handshake):
1. **Client Hello** — клиент отправляет список поддерживаемых TLS-версий и cipher suites.
2. **Server Hello** — сервер выбирает TLS-версию (актуально TLS 1.3 — быстрее, 1 round-trip).
3. **Сертификат сервера** — клиент проверяет цепочку доверия (CA → intermediate → server).
4. **Key exchange** — стороны договариваются о session keys.
5. **Encrypted communication** — все данные шифруются.

В KMP вы обычно не работаете с TLS напрямую — engine (OkHttp, Darwin, Java) обрабатывает это. Но в продакшене нужно знать о трёх аспектах: certificate pinning, custom CA, и поддержка HTTP/3 (QUIC).

### Certificate Pinning — защита от MITM

**Проблема:** по умолчанию клиент доверяет любому сертификату, подписанному известным CA. Если CA скомпрометирован (случалось с DigiNotar в 2011, Symantec в 2018) или на устройстве установлен пользовательский root-CA (корпоративный прокси, рутованное устройство) — атакующий может перехватить трафик (Man-in-the-Middle).

**Решение:** certificate pinning — клиент дополнительно проверяет, что серверный сертификат соответствует ожидаемому public key. Даже если CA скомпрометирован, MITM не пройдёт.

#### Pinning через OkHttp (Android, Desktop JVM)

```kotlin
import okhttp3.CertificatePinner

val pinner = CertificatePinner.Builder()
    .add("api.example.com", "sha256/47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=")
    .add("api.example.com", "sha256/mB0YQ8j5GE6cYzZJq9ZPeU3FYp3/o5E+p9Xk4Fr2jcI=")  // backup pin
    .build()

val client = HttpClient(OkHttp) {
    engine {
        config {
            certificatePinner(pinner)
        }
    }
}
```

> **Важно:** всегда добавляйте **backup pin** (второй публичный ключ). Если основной ключ скомпрометирован или истекает, вы сможете переключиться на backup без выпуска нового приложения.

#### Pinning через Android NetworkSecurityConfig (XML)

Альтернатива — настроить pinning на уровне Android-манифеста, без изменения Kotlin-кода:

`androidApp/src/main/res/xml/network_security_config.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2027-12-31">
            <pin digest="SHA-256">47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFuFU=</pin>
            <pin digest="SHA-256">mB0YQ8j5GE6cYzZJq9ZPeU3FYp3/o5E+p9Xk4Fr2jcI=</pin>  <!-- backup -->
        </pin-set>
    </domain-config>
</network-security-config>
```

Подключение в `AndroidManifest.xml`:
```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ...>
</application>
```

#### Pinning на iOS (Darwin engine)

На iOS certificate pinning реализуется через `URLSessionDelegate` — Darwin engine позволяет передать кастомный delegate:

```kotlin
// iosMain
actual fun createHttpClient(): HttpClient = HttpClient(Darwin) {
    engine {
        configureSession {
            sessionDelegate = PinningDelegate()
        }
    }
}

class PinningDelegate : NSObject(), NSURLSessionDelegate {
    override fun URLSession(
        session: NSURLSession,
        didReceiveChallenge: NSURLAuthenticationChallenge,
        completionHandler: (NSURLSessionAuthChallengeDisposition, NSURLCredential?) -> Unit
    ) {
        val serverTrust = didReceiveChallenge.protectionSpace.serverTrust
        if (serverTrust != null) {
            val cert = SecTrustCopyCertificateChain(serverTrust)?.firstOrNull()
            val publicKey = SecCertificateCopyKey(cert)
            // Сравниваем publicKey с ожидаемым
            if (publicKey == expectedPublicKey) {
                completionHandler(
                    NSURLSessionAuthChallengeUseCredential,
                    NSURLCredential.createTrust(serverTrust)
                )
            } else {
                completionHandler(NSURLSessionAuthChallengeCancelAuthenticationChallenge, null)
            }
        } else {
            completionHandler(NSURLSessionAuthChallengePerformDefaultHandling, null)
        }
    }
}
```

> **Библиотеки для упрощения:** [TrustKit](https://github.com/datatheorem/TrustKit-iOS) — популярная iOS-библиотека для pinning, может быть интегрирована через expect/actual.

### Самоподписанные сертификаты и custom CA

Для разработки (или корпоративных приложений с внутренним CA) нужно доверять нестандартным сертификатам:

```xml
<!-- androidApp/src/main/res/xml/network_security_config.xml -->
<network-security-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="user"/>  <!-- доверять user-installed CA в debug -->
            <certificates src="system"/>
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

```kotlin
// iosMain — через Darwin engine (ТОЛЬКО для debug!)
engine {
    configureSession {
        // URLSessionDelegate с allowSelfSignedCertificates = true
    }
}
```

> **КРИТИЧНО:** никогда не отправляйте debug-конфигурацию с self-signed-доверием в production. Это полностью отключает защиту TLS.

### HTTP/3 и QUIC

**HTTP/3** — следующее поколение HTTP, работает поверх **QUIC** (Quick UDP Internet Connections) вместо TCP. Преимущества:
- **0-RTT handshake** — при повторном подключении данные отправляются сразу, без рукопожатия.
- **Multiplexing без head-of-line blocking** — несколько запросов в одном соединении, потеря пакета одного запроса не блокирует другие.
- **Connection migration** — соединение сохраняется при переключении WiFi↔LTE.

#### Поддержка HTTP/3 в KMP-стеке (на 30 июня 2026)

| Engine | HTTP/3 статус | Путь включения |
|--------|---------------|----------------|
| **OkHttp** (Android) | ✅ через Cronet / experimental | `okhttp3.Http3` (experimental), или оборачивание Google Cronet |
| **Darwin** (iOS) | ✅ нативно (NSURLSession) | Включается автоматически, если сервер поддерживает |
| **Java 11+** (Desktop) | ❌ | Используйте OkHttp или Jetty HTTP3 client |
| **CIO** (Ktor) | ❌ HTTP/1.x only | — |
| **Js** (Wasm) | ❌ | Браузерный Fetch API ещё не стабилизировал HTTP/3 |

> **Важно (актуально на Ktor 3.4.2):** Ktor CIO engine — **только HTTP/1.x**. Для HTTP/3 используйте OkHttp (с Cronet adapter) на Android или Darwin engine на iOS — они автоматически negotiate на HTTP/3 если сервер поддерживает (через Alt-Svc header).

#### Включение HTTP/3 через OkHttp + Cronet

```kotlin
// build.gradle.kts (androidMain)
dependencies {
    implementation("com.google.android.gms:play-services-cronet:18.1.0")
    implementation("com.squareup.okhttp3:okhttp:5.0.0-alpha.14")  // или 4.12+ с экспериментальной поддержкой
}
```

```kotlin
// androidMain
import okhttp3.OkHttpClient
import org.chromium.net.CronetEngine

val cronetEngine = CronetEngine.Builder(context)
    .enableHttpCache(CronetEngine.Builder.HTTP_CACHE_IN_MEMORY, 1024 * 1024)
    .enableHttp2(true)
    .enableQuic(true)  // включает HTTP/3
    .build()

val okHttpClient = OkHttpClient.Builder()
    // адаптер Cronet → OkHttp
    .addInterceptor(CronetInterceptor.newBuilder(context).setCronetEngine(cronetEngine).build())
    .build()

val client = HttpClient(OkHttp) {
    engine { preconfigured = okHttpClient }
}
```

#### Когда выбирать HTTP/3

✅ **Используйте HTTP/3, если:**
- Мобильное приложение с частыми короткими запросами (мессенджеры, трейдинг).
- Сервер за CDN с поддержкой HTTP/3 (Cloudflare, Fastly, Google Cloud CDN).
- Пользователи часто переключаются между сетями (WiFi↔LTE).

❌ **Не используйте HTTP/3, если:**
- Сервер не поддерживает QUIC — клиенты автоматически откатятся на HTTP/2, но это лишняя работа.
- Приложение работает в закрытой сети без UDP (некоторые корпоративные firewall блокируют UDP).
- Нужна максимальная совместимость — HTTP/2 до сих пор более распространён.

> **Best practice:** Включайте HTTP/3 как опциональную оптимизацию. Если клиент не может установить QUIC-соединение, он должен прозрачно откатиться на HTTP/2 или HTTP/1.1. Не делайте HTTP/3 обязательным требованием.

---

## 6.8. Best practices

| Сценарий | Решение |
|----------|---------|
| **Сетевые ошибки** | Retry с экспоненциальной задержкой, fallback на кэш |
| **HTTP 401** | Автоматическое обновление токена через `refreshTokens` |
| **HTTP 422 (Validation)** | Десериализация тела ошибки в `ValidationErrorResponse`, показ ошибок пользователю |
| **HTTP 5xx** | Retry, затем fallback на кэш с индикатором «данные могут быть устаревшими» |
| **Таймаут** | Настроить в `HttpTimeout` (30s request, 10s connect) |
| **Forward-compatibility** | `ignoreUnknownKeys = true` в Json |
| **Логирование в debug** | `Logging` плагин с `LogLevel.HEADERS`, отключить в release |
| **Безопасность** | HTTPS-only, certificate pinning для критичных API, network security config |
| **Performance для real-time** | HTTP/3 (QUIC) через OkHttp+Cronet на Android, Darwin на iOS |

---

⬅️ [← Работа с данными](05-data-storage.md) | [К содержанию](README.md) | [Локализация →](07-localization.md) ➡️
