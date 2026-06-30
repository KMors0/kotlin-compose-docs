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
- **Google Tink** — высокоуровневая крипто-библиотека от Google, упрощает правильное использование криптографии.
- **Платформенные API:** Keychain (iOS), Android Keystore.

**Пример шифрования строки (через javax.crypto):**
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

## 12.3. Google Tink — высокоуровневая криптография

[Google Tink](https://github.com/tink-crypto/tink) — кроссплатформенная open-source крипто-библиотека от Google. Главное преимущество — **безопасность по умолчанию**: библиотека скрывает опасные низкоуровневые детали (выбор IV, padding, режим шифрования) и предоставляет простой API, в котором сложно ошибиться.

> [!TIP]
> **Зачем Tink, если есть javax.crypto?**
> - `javax.crypto` доступен только на JVM/Android. На iOS, Web, Kotlin/Native его нет.
> - Tink — мультиязыковой (Java, C++, Go, Python, JavaScript), работает везде.
> - Tink использует ключи, помеченные как "primary", "rotated", "disabled" — упрощает ротацию ключей без прерывания работы приложения.
> - Tink генерирует безопасные IV/nonce автоматически, вам не нужно думать об этом.

### Подключение

```toml
# gradle/libs.versions.toml
[versions]
tink = "1.16.0"

[libraries]
tink = { module = "com.google.crypto.tink:tink", version.ref = "tink" }
```

```kotlin
// build.gradle.kts (androidMain или commonMain если KMP-обёртка)
dependencies {
    implementation(libs.tink)
}
```

> **Важно (на 30 июня 2026):** Tink официально поддерживает Android и JVM. Для iOS/Kotlin-Native нужно либо использовать платформенные CryptoKit/CommonCrypto через expect/actual, либо обернуть Tink в Kotlin/JVM-сервис на сервере. Для полностью кроссплатформенного шифрования в KMP-проекте можно использовать [kotlin-multiplatform-libsodium](https://github.com/ionspin/kotlin-multiplatform-libsodium).

### Симметричное шифрование (AES-GCM)

```kotlin
import com.google.crypto.tink.KeysetHandle
import com.google.crypto.tink.aead.AeadConfig
import com.google.crypto.tink.aead.AesGcmKeyManager
import com.google.crypto.tink.aead.AesGcmHkdfStreamingKeyManager
import com.google.crypto.tink.KeyTemplates

// 1. Регистрируем AEAD (аутентифицированное шифрование)
AeadConfig.register()

// 2. Генерируем новый ключ (256-bit AES-GCM)
val keysetHandle: KeysetHandle = KeysetHandle.generateNew(
    KeyTemplates.get("AES256_GCM")
)

// 3. Получаем AEAD-примитив
val aead = keysetHandle.getPrimitive(Aead::class.java)

// 4. Шифруем
val plaintext = "Секретное сообщение".toByteArray()
val associatedData = "user:alice".toByteArray()  // привязка к контексту
val ciphertext: ByteArray = aead.encrypt(plaintext, associatedData)

// 5. Дешифруем
val decrypted: ByteArray = aead.decrypt(ciphertext, associatedData)
println(String(decrypted))  // "Секретное сообщение"
```

> [!TIP]
> **`associatedData` (AAD) в AEAD-шифровании** — дополнительные данные, которые не шифруются, но **аутентифицируются**. Если AAD изменится при дешифровке — операция завершится с ошибкой. Используется для привязки шифртекста к контексту: например, `associatedData = "user:alice"` гарантирует, что шифртекст, созданный для Alice, нельзя расшифровать как сообщение для Bob.

### Хранение ключей: Keyset

Tink хранит ключи в **Keyset** — наборе ключей с метаданными (primary, rotated, disabled). Keyset можно сериализовать в JSON или бинарный формат:

```kotlin
import com.google.crypto.tink.CleartextKeysetHandle
import com.google.crypto.tink.JsonKeysetWriter
import java.io.ByteArrayOutputStream

// Сериализация keyset в JSON (для хранения в Keychain/Keystore)
val outputStream = ByteArrayOutputStream()
CleartextKeysetHandle.write(keysetHandle, JsonKeysetWriter.withOutputStream(outputStream))
val keysetJson: String = outputStream.toString()

// Внимание: keysetJson содержит ключи в открытом виде!
// Храните его только в защищённом хранилище (Android Keystore, iOS Keychain).
```

### Ротация ключей

```kotlin
import com.google.crypto.tink.KeysetManager

// Добавляем новый ключ и делаем его primary
val rotatedKeyset = KeysetManager.withKeysetHandle(keysetHandle)
    .rotate(KeyTemplates.get("AES256_GCM"))
    .keysetHandle

// Старый ключ остаётся в keyset для дешифровки старых данных,
// но новые данные шифруются новым primary ключом.
```

Ротация ключей — best practice: если ключ скомпрометирован, ущерб ограничен данными, зашифрованными этим ключом. Tink делает ротацию прозрачной — вам не нужно менять код.

### Гибридное шифрование (ECIES)

Для сценариев типа «отправить зашифрованное сообщение, не имея предварительного общего ключа» (мессенджеры):

```kotlin
import com.google.crypto.tink.HybridConfig
import com.google.crypto.tink.hybrid.HybridKeyTemplates
import com.google.crypto.tink.hybrid.HybridEncrypt
import com.google.crypto.tink.hybrid.HybridDecrypt

HybridConfig.register()

// Получатель генерирует пару ключей
val recipientKeyset = KeysetHandle.generateNew(HybridKeyTemplates.get("DHKEM_X25519_HKDF_SHA256_HKDF_SHA256_AES_256_GCM"))
val publicKey = recipientKeyset.getPublicKey()

// Отправитель шифрует публичным ключом
val hybridEncrypt = publicKey.getPrimitive(HybridEncrypt::class.java)
val ciphertext = hybridEncrypt.encrypt(plaintext, contextInfo)

// Получатель дешифрует приватным ключом
val hybridDecrypt = recipientKeyset.getPrimitive(HybridDecrypt::class.java)
val decrypted = hybridDecrypt.decrypt(ciphertext, contextInfo)
```

> [!TIP]
> **Когда использовать гибридное шифрование:** в P2P-мессенджерах (Direct-talk, E2E-чат) — когда две стороны не имеют общего ключа заранее, но хотят обмениваться зашифрованными сообщениями. См. [главу 38. P2P-сети и мессенджеры](38-p2p-networking.md) для полного руководства.

### Сравнение крипто-решений для KMP

| Библиотека | Платформы | Сложность | Когда выбирать |
|------------|-----------|-----------|----------------|
| **Google Tink** | JVM, Android (полностью); iOS — через обёртки | Низкая (high-level API) | Production-крипто на Android/JVM, простая ротация ключей |
| **libsodium (kotlin-multiplatform-libsodium)** | Все KMP-платформы | Средняя | Полностью кроссплатформенное шифрование, включая iOS/Wasm |
| **Bouncy Castle** | JVM, Android | Высокая (низкоуровневый API) | Нестандартные алгоритмы, совместимость с legacy-системами |
| **javax.crypto** | JVM, Android | Средняя | Простые сценарии, когда не нужна кроссплатформенность |
| **CryptoKit (iOS)** | iOS только | Низкая | Native iOS-приложения без KMP |
| **Web Crypto API** | Web (Wasm/JS) | Низкая | Web-таргет |

> **Best practice для KMP-мессенджера:** используйте **kotlin-multiplatform-libsodium** для полностью кроссплатформенного шифрования (Android, iOS, Desktop, Wasm). Tink — отличный выбор, если нужно серверное шифрование на JVM или Android-only клиент.

---

## 12.4. Защита от уязвимостей

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
