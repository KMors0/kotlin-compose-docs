# Безопасность

> Безопасность — критически важный аспект любого приложения, работающего с пользовательскими данными. В этой части рассматриваются подходы к хранению секретов, шифрованию данных и защите от распространённых уязвимостей.

---

## 12.1. Хранение секретов

- **Gradle Properties:** храните API-ключи в `gradle.properties`.
- **Облачные менеджеры:** AWS Secrets Manager, Google Secret Manager.
- **.env файлы:** используйте библиотеки вроде `dotenv` для загрузки переменных.

Никогда не храните секреты (API-ключи, токены, пароли) непосредственно в исходном коде. Даже если репозиторий закрытый, утечка ключей — лишь вопрос времени. Рекомендуется использовать облачные менеджеры секретов для production-окружения и локальные файлы `.env` (исключённые из VCS через `.gitignore`) для разработки. Gradle Properties позволяет внедрять секреты на этапе сборки через `BuildConfig`.

**Пример с BuildConfig:**
```kotlin
// build.gradle.kts
android {
    defaultConfig {
        buildConfigField("String", "API_KEY", "\"${project.property("api_key")}\"")
    }
}

// Использование
val apiKey = BuildConfig.API_KEY
```

---

## 12.2. Шифрование данных

- **Bouncy Castle** — кроссплатформенная криптографическая библиотека.
- **Платформенные API:** Keychain (iOS), Android Keystore.

**Пример шифрования строки:**
```kotlin
val cipher = Cipher.getInstance("AES/GCM/NoPadding")
cipher.init(Cipher.ENCRYPT_MODE, secretKey)
val encrypted = cipher.doFinal(data.toByteArray())
```

Для кроссплатформенного шифрования используйте Bouncy Castle — зрелую криптографическую библиотеку с поддержкой Kotlin Multiplatform. Для хранения ключей шифрования на каждой платформе есть безопасные хранилища: Android Keystore (аппаратное хранение ключей) и iOS Keychain (защищённое хранилище). Используйте `expect/actual` для абстракции платформенных хранилищ ключей.

**Пример expect/actual для безопасного хранения ключей:**
```kotlin
// Общий модуль
expect class SecureKeyStore {
    fun getKey(alias: String): SecretKey
    fun storeKey(alias: String, key: SecretKey)
}

// Android
actual class SecureKeyStore {
    actual fun getKey(alias: String): SecretKey {
        val keyStore = KeyStore.getInstance("AndroidKeyStore")
        return keyStore.getKey(alias, null) as SecretKey
    }
}
```

---

## 12.3. Защита от уязвимостей

- **SQL-инъекции:** используйте параметризованные запросы (SQLDelight).
- **XSS:** санитизируйте пользовательский ввод в веб-версии.
- **TLS:** всегда используйте защищённые соединения (HTTPS).

SQL-инъекции остаются одной из самых распространённых уязвимостей. SQLDelight автоматически параметризует все запросы, что полностью исключает возможность SQL-инъекций. Для веб-версии приложения обязательно санитизируйте любой пользовательский ввод перед отображением в DOM — Kotlin/JS не имеет встроенной защиты от XSS. Все сетевые запросы должны выполняться только через HTTPS; при необходимости добавьте certificate pinning.

**Чеклист безопасности:**
- [ ] Секреты не хранятся в исходном коде
- [ ] Используются параметризованные SQL-запросы
- [ ] Сетевые запросы идут только через HTTPS
- [ ] Пользовательский ввод санитизируется
- [ ] Чувствительные данные шифруются при хранении
- [ ] Ключи шифрования хранятся в платформенных хранилищах (Keystore/Keychain)
- [ ] Реализована защита от перехвата токенов (short-lived tokens, refresh rotation)

---

⬅️ [← Платформенная интеграция](11-platform-integration.md) | [К содержанию](README.md) | [DevOps и CI/CD →](13-devops-cicd.md) ➡️
