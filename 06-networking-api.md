# Интеграция с API и сетевые запросы

> Сетевое взаимодействие — неотъемлемая часть большинства современных приложений. В этой части рассматривается работа с HTTP-клиентом Ktor, обработка JSON, обработка ошибок и аутентификация в кроссплатформенном контексте.

---

## 6.1. Ktor

Ktor — мультиплатформенная HTTP-библиотека.

**Подключение:**
```kotlin
implementation("io.ktor:ktor-client-core:$ktor_version")
implementation("io.ktor:ktor-client-okhttp:$ktor_version") // Android
implementation("io.ktor:ktor-client-cio:$ktor_version") // iOS/Web
```

**Пример запроса:**
```kotlin
suspend fun fetchTasks(): List<Task> {
    val client = HttpClient(OkHttp)
    return client.get("https://api.example.com/tasks")
}
```

Ktor — это фреймворк от JetBrains, который включает как серверную, так и клиентскую часть. Для мобильной разработки нас интересует `ktor-client` — мультиплатформенный HTTP-клиент, который работает на Android (через OkHttp), iOS (через Darwin) и Web (через JS Fetch API). Архитектура Ktor основана на системе плагинов, что позволяет добавлять только нужный функционал — сериализацию, логирование, аутентификацию.

---

## 6.2. Обработка JSON

Используйте `ktor-client-serialization` с `kotlinx.serialization`.

```kotlin
@Serializable
data class TaskResponse(val tasks: List<Task>)

val client = HttpClient {
    install(JsonFeature) {
        serializer = KotlinxSerializer()
    }
}

suspend fun getTasks(): List<Task> = client.get<TaskResponse>("https://api.example.com/tasks").tasks
```

`kotlinx.serialization` — официальный фреймворк сериализации от JetBrains. В отличие от Gson или Jackson, он работает на всех платформах, поддерживаемых Kotlin Multiplatform, и генерирует код сериализации на этапе компиляции. Это означает отсутствие рефлексии в runtime, что особенно важно для Kotlin/Native (iOS), где рефлексия ограничена.

---

## 6.3. Обработка ошибок

- Используйте `try-catch` для сетевых исключений.
- Реализуйте `RetryPolicy` для временных ошибок.
- Кэшируйте ответы для улучшения UX.

**Пример:**
```kotlin
try {
    val response = client.get(...)
} catch (e: ClientRequestException) {
    // обработка HTTP-ошибок (4xx)
} catch (e: ServerResponseException) {
    // обработка серверных ошибок (5xx)
} catch (e: Exception) {
    // общая обработка
}
```

Стратегия обработки ошибок должна включать несколько уровней. На первом уровне — перехват конкретных HTTP-ошибок (404, 401, 500). На втором уровне — повторные попытки для временных ошибок (таймаут, отсутствие сети). На третьем уровне — кэширование последних успешных ответов для отображения данных даже при отсутствии соединения. Ktor предоставляет встроенные плагины для реализации retry-логики и кэширования.

---

## 6.4. Аутентификация

- **Bearer Token:** передавайте токен в заголовке `Authorization`.
- **OAuth2:** используйте `ktor-client-auth`.

**Пример Bearer Token:**
```kotlin
client.config {
    defaultRequest {
        header("Authorization", "Bearer $token")
    }
}
```

**Пример с плагином аутентификации:**
```kotlin
val client = HttpClient {
    install(Auth) {
        bearer {
            loadTokens {
                BearerTokens(accessToken = "token", refreshToken = "refresh")
            }
            refreshTokens {
                BearerTokens(accessToken = "new_token", refreshToken = "new_refresh")
            }
        }
    }
}
```

Плагин `Auth` в Ktor автоматически добавляет токен к каждому запросу и обрабатывает обновление токена при истечении срока действия. Это значительно упрощает работу с защищёнными API — вам не нужно вручную отслеживать срок действия токена и повторять запросы.

---

⬅️ [← Работа с данными](05-data-storage.md) | [К содержанию](README.md) | [Локализация →](07-localization.md) ➡️
