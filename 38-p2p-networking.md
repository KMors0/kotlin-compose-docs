# P2P-сети и мессенджеры

> Эта глава — для тех, кто строит децентрализованные приложения: мессенджеры с end-to-end шифрованием, P2P-чаты, гибридные архитектуры (сервер + peer-to-peer). Здесь разобраны QUIC, WebRTC, STUN/TURN/ICE, сигналинг, NAT traversal, синхронизация между устройствами и архитектурные паттерны. Все примеры актуальны для Kotlin Multiplatform 1.11 и Ktor 3.4.

---

## 38.1. Архитектуры сетевого взаимодействия

Сетевые приложения делятся на три основные архитектуры. Понимание различий — ключ к выбору правильного стека.

### Клиент-сервер (централизованная)

```
[Клиент A] ──→ [Сервер] ←── [Клиент B]
```

Все сообщения проходят через сервер. Сервер хранит историю, аутентифицирует пользователей, маршрутизирует сообщения.

**Примеры:** Telegram, WhatsApp (классический), Slack, Discord.

| Плюсы | Минусы |
|-------|--------|
| Простая реализация | Сервер — единая точка отказа |
| История сообщений доступна с любого устройства | Сервер видит весь трафик (нет E2E по умолчанию) |
| Работает через любой NAT | Затраты на инфраструктуру |
| Push-уведомления из коробки | Юридические обязательства (хранение данных по требованию регуляторов) |

### P2P (peer-to-peer, децентрализованная)

```
[Клиент A] ←──→ [Клиент B]
```

Клиенты соединяются напрямую, без сервера-посредника. Сервер (если есть) используется только для первоначального обмена контактами (сигналинг).

**Примеры:** Tox, Briar, Direct-talk в ReMatch Chat.

| Плюсы | Минусы |
|-------|--------|
| Нет центрального сервера — невозможно отключить | Сложность обхода NAT |
| Сервер не видит трафик (полное E2E) | История сообщений только на устройствах |
| Нет затрат на инфраструктуру | Сложность синхронизации между устройствами |
| Лучшая приватность | Push-уведомления требуют отдельного решения |

### Гибридная (ReMatch Chat, Signal, Matrix)

```
[Клиент A] ──→ [Сигналинг-сервер] ←── [Клиент B]
                    ↓ (только для установления соединения)
[Клиент A] ←═══════ P2P / через посредника ═══════→ [Клиент B]
```

Сервер помогает установить соединение (обмен публичными ключами, STUN/TURN для NAT traversal), но сами сообщения идут напрямую между клиентами, зашифрованные E2E.

**Примеры:** Signal (сигналинг + E2E), Matrix (федерация + E2E), ReMatch Chat (Хаб + Direct-talk).

| Плюсы | Минусы |
|-------|--------|
| E2E-шифрование | Сложнее в реализации |
| Push-уведомления через Хаб | Нужно два стека технологий (сервер + P2P) |
| Резервные пути через Посредника | Синхронизация ключей между устройствами |
| Соблюдение законов (Хаб хранит только публичные данные) | Юридические вопросы вокруг Посредников |

---

## 38.2. QUIC — транспортный протокол нового поколения

**QUIC** (Quick UDP Internet Connections) — транспортный протокол от Google, стандартизированный в RFC 9000 (2021). Лежит в основе HTTP/3. Использует UDP вместо TCP.

> [!TIP]
> **QUIC vs TCP — что лучше для мессенджера?**
> - **TCP** — надёжная доставка, но head-of-line blocking: потеря одного пакета блокирует все последующие.
> - **QUIC** — несколько независимых потоков в одном соединении. Потеря пакета в одном потоке не блокирует другие.
> - **QUIC** — 0-RTT handshake при повторном подключении (мгновенное восстановление связи).
> - **QUIC** — connection migration: при переключении WiFi↔LTE соединение сохраняется.

### Поддержка QUIC в Kotlin Multiplatform

| Платформа | Библиотека | Статус |
|-----------|------------|--------|
| **Android** | Cronet (Google Play Services) | ✅ Stable |
| **Android** | OkHttp + HTTP/3 plugin | ✅ Experimental (OkHttp 5.x) |
| **iOS** | NSURLSession (Darwin engine в Ktor) | ✅ Native |
| **Desktop (JVM)** | Cronet + Java wrapper | ⚠️ Требует JNI-обёртку |
| **Web (Wasm)** | Браузерный Fetch API | ⚠️ Зависит от браузера |

### Пример QUIC-клиента через Cronet (Android)

```kotlin
import org.chromium.net.CronetEngine
import org.chromium.net.CronetException
import org.chromium.net.UrlRequest
import org.chromium.net.UrlResponseInfo
import java.nio.ByteBuffer
import java.util.concurrent.Executor

class QuicClient(private val context: Context) {

    private val executor: Executor = Executors.newSingleThreadExecutor()

    private val cronetEngine: CronetEngine = CronetEngine.Builder(context)
        .enableQuic(true)              // включаем QUIC
        .enableHttp2(true)             // fallback на HTTP/2
        .enableBrotli(true)            // сжатие ответов
        .enableHttpCache(
            CronetEngine.Builder.HTTP_CACHE_IN_MEMORY,
            1024 * 1024  // 1 МБ кэш
        )
        .build()

    fun sendMessage(url: String, body: ByteArray, callback: (ByteArray?, Exception?) -> Unit) {
        val request = cronetEngine.newUrlRequestBuilder(
            url,
            object : UrlRequest.Callback() {
                private val responseBuffer = ByteArrayOutputStream()

                override fun onRedirectReceived(
                    request: UrlRequest,
                    info: UrlResponseInfo,
                    newLocationUrl: String
                ) {
                    request.followRedirect()
                }

                override fun onResponseStarted(request: UrlRequest, info: UrlResponseInfo) {
                    request.read(ByteBuffer.allocateDirect(32 * 1024))
                }

                override fun onReadCompleted(
                    request: UrlRequest,
                    info: UrlResponseInfo,
                    byteBuffer: ByteBuffer
                ) {
                    byteBuffer.flip()
                    val bytes = ByteArray(byteBuffer.remaining())
                    byteBuffer.get(bytes)
                    responseBuffer.write(bytes)
                    byteBuffer.clear()
                    request.read(byteBuffer)
                }

                override fun onSucceeded(request: UrlRequest, info: UrlResponseInfo) {
                    callback(responseBuffer.toByteArray(), null)
                }

                override fun onFailed(
                    request: UrlRequest,
                    info: UrlResponseInfo?,
                    error: CronetException
                ) {
                    callback(null, error)
                }
            },
            executor
        ).setHttpMethod("POST").addHeader("Content-Type", "application/octet-stream").build()

        // Тело запроса
        val bodyProvider = UrlRequest.Builder()
        // Cronet требует UploadDataProvider для тела — упрощено для примера
        request.start()
    }
}
```

> [!TIP]
> **Cronet — единственный production-ready способ использовать QUIC на Android**. Альтернативы:
> - **OkHttp 5.x** — экспериментальная поддержка HTTP/3, но нестабильная.
> - **Ktor CIO engine** — НЕ поддерживает QUIC (только HTTP/1.x на 3.4.2).
> - **java.net.http.HttpClient** (Java 11+) — НЕ поддерживает QUIC.

### Прямое QUIC-соединение (UDP-based)

QUIC работает поверх UDP. Для P2P-соединений можно использовать сырой UDP через `kotlinx.coroutines` + платформенные сокеты:

```kotlin
// commonMain
expect class QuicConnection(remoteAddress: String, remotePort: Int) {
    suspend fun connect(): Boolean
    suspend fun send(data: ByteArray): Int
    suspend fun receive(): ByteArray
    fun close()
}

// androidMain — через DatagramChannel (UDP)
actual class QuicConnection actual constructor(
    private val remoteAddress: String,
    private val remotePort: Int
) {
    private val channel = DatagramChannel.open()
        .configureBlocking(false)
        .bind(null)  // случайный локальный порт

    actual suspend fun connect(): Boolean {
        // QUIC handshake — упрощённо, в реальности используйте quiche или lsquic
        val addr = InetSocketAddress(remoteAddress, remotePort)
        channel.connect(addr)
        return channel.isConnected
    }

    actual suspend fun send(data: ByteArray): Int {
        val buffer = ByteBuffer.wrap(data)
        return channel.send(buffer, InetSocketAddress(remoteAddress, remotePort))
    }

    actual suspend fun receive(): ByteArray {
        val buffer = ByteBuffer.allocate(65535)
        channel.receive(buffer)
        buffer.flip()
        return ByteArray(buffer.remaining()).also { buffer.get(it) }
    }

    actual fun close() = channel.close()
}
```

> [!TIP]
> Для production P2P-QUIC рассматривайте **[quiche](https://github.com/cloudflare/quiche)** от Cloudflare или **[lsquic](https://github.com/litespeedtech/lsquic)** — обе библиотеки можно обернуть через JNI (Android) и cinterop (iOS). См. [главу 11.6. Работа с нативным кодом](11-platform-integration.md#116-работа-с-нативным-кодом-jni-c-interop).

---

## 38.3. NAT Traversal: STUN, TURN, ICE

Большинство устройств за NAT (Network Address Translation) — домашние роутеры, корпоративные сети, мобильные операторы. Два устройства за NAT не могут соединиться напрямую: они не знают свои реальные публичные адреса.

### STUN — обнаружение публичного адреса

**STUN** (Session Traversal Utilities for NAT) — протокол, который помогает клиенту узнать свой **публичный IP и порт**. Клиент отправляет запрос на STUN-сервер, тот отвечает: «я вижу тебя как 203.0.113.42:54321».

```
[Клиент A за NAT] ──→ [STUN сервер]
[Клиент A за NAT] ←── "Твой публичный адрес: 203.0.113.42:54321"
```

После этого клиент может сообщить свой публичный адрес другому клиенту через сигналинг-сервер.

> [!TIP]
> **STUN работает не со всеми типами NAT.** Симметричный NAT (symmetric NAT) — самый проблемный: для каждого получателя назначается разный публичный порт. STUN не поможет, нужно использовать TURN.

### TURN — ретрансляция трафика

**TURN** (Traversal Using Relays around NAT) — сервер-посредник, который ретранслирует трафик между двумя клиентами, если прямое P2P-соединение невозможно.

```
[Клиент A] ──→ [TURN сервер] ←── [Клиент B]
```

TURN — fallback, когда STUN не сработал. Трафик идёт через сервер, но **зашифрован E2E** — сервер видит только зашифрованные байты.

> [!TIP]
> **TURN не нарушает E2E-шифрование.** Сервер TURN — это просто ретранслятор (relay), он не расшифровывает трафик. Клиенты шифруют сообщения своими ключами, TURN пересылает зашифрованные байты. Это безопасно — даже владелец TURN-сервера не может прочитать сообщения.

### ICE — оркестрация

**ICE** (Interactive Connectivity Establishment) — алгоритм, который перебирает все возможные пути соединения и выбирает лучший:

1. **Host candidate** — прямой локальный адрес (если клиенты в одной LAN).
2. **Server reflexive candidate** — публичный адрес от STUN.
3. **Relay candidate** — адрес TURN-сервера (если ничего другое не сработало).

ICE перебирает кандидатов параллельно и выбирает первый, по которому установлено соединение. Обычно предпочтение: host > server reflexive > relay (последний — самый медленный из-за ретрансляции).

### Реализация в Ktor WebRTC Client

В Ktor 3.x появился встроенный `WebRtcClient` с поддержкой STUN/TURN:

```kotlin
import io.ktor.client.plugins.webrtc.*

val client = HttpClient {
    install(WebRtc) {
        // STUN-серверы (бесплатные публичные от Google)
        iceServers = listOf(
            IceServer(
                urls = listOf("stun:stun.l.google.com:19302"),
                username = null,
                credential = null
            ),
            // TURN-сервер (нужен свой или платный сервис вроде Twilio)
            IceServer(
                urls = listOf("turn:turn.example.com:3478"),
                username = "user",
                credential = "password"
            )
        )
        iceTransportPolicy = IceTransportPolicy.ALL  // или RELAY для принудительного TURN
    }
}
```

### Самостоятельный TURN-сервер (coturn)

Для production разворачивают собственный TURN-сервер. [coturn](https://github.com/coturn/coturn) — самый популярный open-source вариант:

```bash
# Установка на Ubuntu/Debian
sudo apt install coturn

# Минимальная конфигурация /etc/turnserver.conf
listening-port=3478
tls-listening-port=5349
realm=example.com
server-name=turn.example.com
user=user:password
lt-cred-mech
fingerprint
no-cli
```

```bash
# Запуск
turnserver -c /etc/turnserver.conf --external-ip=203.0.113.42
```

> [!TIP]
> **Разворачивание своего TURN — необходимость для production P2P-приложения.** Бесплатные публичные STUN есть (Google, OpenRelay), но бесплатных TURN мало, и они ненадёжны. Свой TURN-сервер стоит ~$5-20/месяц на VPS — это страховка от «у пользователя не работает звонок за NAT».

---

## 38.4. Сигналинг — установление P2P-соединения

Чтобы два клиента могли установить P2P-соединение, им нужно обменяться информацией:
- Публичные ключи (для E2E-шифрования).
- ICE candidates (адреса для соединения).
- Метаданные сессии (какие кодеки поддерживаются, какой протокол).

Этот обмен называется **сигналингом**. Сигналинг идёт через обычный сервер (HTTPS, WebSocket), а после установления P2P — сервер не нужен.

### Сигналинг через WebSocket (Ktor)

```kotlin
// commonMain — клиент сигналинга
class SignalingClient(
    private val client: HttpClient,
    private val signalingUrl: String  // wss://hub.example.com/signaling
) {
    private val incomingMessages = MutableSharedFlow<SignalingMessage>()

    val messages: SharedFlow<SignalingMessage> = incomingMessages.asSharedFlow()

    suspend fun connect(userId: String) {
        client.webSocket("$signalingUrl?user=$userId") {
            while (true) {
                val message = receiveDeserialized<SignalingMessage>()
                incomingMessages.emit(message)
            }
        }
    }

    suspend fun send(message: SignalingMessage) {
        // Отправка через WebSocket
        client.webSocket(signalingUrl) {
            sendSerialized(message)
        }
    }
}

@Serializable
sealed class SignalingMessage {
    @Serializable data class IceCandidate(val candidate: String, val sdpMLineIndex: Int) : SignalingMessage()
    @Serializable data class SdpOffer(val sdp: String, val from: String, val to: String) : SignalingMessage()
    @Serializable data class SdpAnswer(val sdp: String, val from: String, val to: String) : SignalingMessage()
    @Serializable data class KeyExchange(val publicKey: String, val from: String) : SignalingMessage()
}
```

### Сервер сигналинга (Ktor server)

```kotlin
import io.ktor.server.application.*
import io.ktor.server.routing.*
import io.ktor.server.websocket.*
import io.ktor.websocket.*
import java.util.concurrent.ConcurrentHashMap

fun Application.configureSignaling() {
    install(WebSockets)
    routing {
        webSocket("/signaling") {
            val userId = call.request.queryParameters["user"] ?: return@webSocket close(
                CloseReason(CloseReason.Codes.VIOLATED_POLICY, "No user id")
            )

            // Регистрируем соединение
            connections[userId] = this

            try {
                for (frame in incoming) {
                    if (frame is Frame.Text) {
                        val message = frame.readText()
                        // Парсим сообщение, определяем получателя
                        val signalingMessage = Json.decodeFromString<SignalingMessage>(message)
                        val recipientId = signalingMessage.recipientId

                        // Пересылаем получателю
                        connections[recipientId]?.send(message)
                    }
                }
            } finally {
                connections.remove(userId)
            }
        }
    }
}

val connections = ConcurrentHashMap<String, WebSocketServerSession>()
```

> [!TIP]
> **Сигналинг-сервер хранит минимум данных:** только активные WebSocket-соединения и маршрутизирует сообщения между ними. Сообщения **не сохраняются** на диске — это просто «телефонная станция» для установления P2P. После handshake сервер можно вообще выключить — P2P-соединение продолжит работать.

---

## 38.5. End-to-End шифрование (E2E)

В P2P-мессенджере сообщения должны быть зашифрованы так, чтобы **никто, кроме отправителя и получателя, не мог их прочитать** — ни сигналинг-сервер, ни Посредник (TURN), ни провайдер.

### Double Ratchet Algorithm (Signal Protocol)

Стандарт де-факто для E2E-мессенджеров — алгоритм Double Ratchet, разработанный Signal. Используется в Signal, WhatsApp, Facebook Messenger (Secret Conversations), Matrix (Olm).

**Ключевые свойства:**
- **Forward secrecy** — компрометация текущего ключа не раскрывает прошлые сообщения.
- **Future secrecy** — компрометация текущего ключа не раскрывает будущие сообщения (после следующего обновления ключа).
- **Out-of-order delivery** — сообщения могут приходить в любом порядке, дешифровка всё равно работает.

### Реализация через libsodium (KMP)

Для Kotlin Multiplatform проще всего использовать [kotlin-multiplatform-libsodium](https://github.com/ionspin/kotlin-multiplatform-libsodium):

```toml
# gradle/libs.versions.toml
[versions]
libsodium = "0.9.2"

[libraries]
libsodium-bindings = { module = "com.ionspin.kotlin:multiplatform-crypto-libsodium-bindings", version.ref = "libsodium" }
```

```kotlin
import com.ionspin.kotlin.crypto.box.Box
import com.ionspin.kotlin.crypto.box.PublicKey
import com.ionspin.kotlin.crypto.box.SecretKey
import com.ionspin.kotlin.crypto.util.LibsodiumRandom

class E2EEncryption {

    // Генерация пары ключей (один раз при регистрации)
    fun generateKeyPair(): KeyPair {
        val publicKey = ByteArray(PublicKey.BYTES)
        val secretKey = ByteArray(SecretKey.BYTES)
        Box.keypair(publicKey, secretKey)
        return KeyPair(publicKey, secretKey)
    }

    // Шифрование сообщения для получателя
    fun encrypt(
        message: ByteArray,
        recipientPublicKey: ByteArray,
        senderSecretKey: ByteArray
    ): EncryptedMessage {
        val nonce = LibsodiumRandom.buf(24)  // 24 байта nonce для crypto_box
        val encrypted = Box.easy(message, nonce, PublicKey(recipientPublicKey), SecretKey(senderSecretKey))
        return EncryptedMessage(ciphertext = encrypted, nonce = nonce)
    }

    // Дешифрование
    fun decrypt(
        encryptedMessage: EncryptedMessage,
        senderPublicKey: ByteArray,
        recipientSecretKey: ByteArray
    ): ByteArray {
        return Box.openEasy(
            encryptedMessage.ciphertext,
            encryptedMessage.nonce,
            PublicKey(senderPublicKey),
            SecretKey(recipientSecretKey)
        )
    }
}

data class KeyPair(val publicKey: ByteArray, val secretKey: ByteArray)
data class EncryptedMessage(val ciphertext: ByteArray, val nonce: ByteArray)
```

> [!TIP]
> **`crypto_box` (X25519+XSalsa20+Poly1305) в libsodium** — это комбинация трёх алгоритмов:
> - **X25519** — обмен ключами (Diffie-Hellman на эллиптических кривых).
> - **XSalsa20** — симметричное шифрование.
> - **Poly1305** — MAC (аутентификация).
>
> Это даёт **E2E-шифрование без предварительного обмена общим ключом** — достаточно обменяться публичными ключами (через сигналинг).

### Идентификация контактов (Safety numbers)

Чтобы убедиться, что публичный ключ собеседника не подменён (MITM-атака), мессенджеры используют **safety numbers** (или fingerprints):

```kotlin
// Вычисление safety number из двух публичных ключей
fun computeSafetyNumber(myPublicKey: ByteArray, theirPublicKey: ByteArray): String {
    // Сортируем ключи лексикографически, чтобы обе стороны получили одинаковое число
    val (first, second) = if (myPublicKey.contentEquals(theirPublicKey)) {
        myPublicKey to theirPublicKey
    } else if (myPublicKey.compareTo(theirPublicKey) < 0) {
        myPublicKey to theirPublicKey
    } else {
        theirPublicKey to myPublicKey
    }

    val combined = first + second
    val hash = SHA256.digest(combined)  // 32 байта
    // Кодируем как 12 групп по 5 цифр (как в Signal)
    return hash.take(30).chunked(5).joinToString(" ") { group ->
        group.joinToString("") { byte -> "%02d".format(byte % 100) }
    }
}
```

Пользователи сравнивают safety number через безопасный канал (лично, по телефону) — если совпадает, значит ключи не подменены.

---

## 38.6. Синхронизация между устройствами

В мессенджере пользователь часто хочет пользоваться аккаунтом с нескольких устройств. В отличие от Telegram (где сервер хранит историю), в P2P-мессенджере синхронизация — сложная задача.

### Вариант 1: через Посредника (как в ReMatch Chat)

Посредник — доверенный сервер (или своё устройство), которое хранит зашифрованную историю. Новое устройство подключается к Посреднику и скачивает зашифрованный архив:

```kotlin
class SyncService(private val mediator: MediatorClient) {

    // Отправка зашифрованной истории на Посредника
    suspend fun backupToMediator(history: List<Message>, encryptionKey: ByteArray) {
        val plaintext = Json.encodeToString(history).toByteArray()
        val compressed = LZ4.compress(plaintext)  // см. раздел 38.7
        val encrypted = AesGcm.encrypt(compressed, encryptionKey)

        mediator.upload(encrypted)
    }

    // Восстановление на новом устройстве
    suspend fun restoreFromMediator(encryptionKey: ByteArray): List<Message> {
        val encrypted = mediator.download()
        val compressed = AesGcm.decrypt(encrypted, encryptionKey)
        val plaintext = LZ4.decompress(compressed)
        return Json.decodeFromString(plaintext.decodeToString())
    }
}
```

### Вариант 2: локальная синхронизация (mDNS / USB)

В одной локальной сети устройства могут найти друг друга через **mDNS** (Bonjour/Avahi) и синхронизироваться напрямую:

```kotlin
// commonMain
expect class MdnsDiscovery() {
    fun startDiscovery(serviceType: String, onFound: (String) -> Unit)
    fun stopDiscovery()
    fun advertise(serviceType: String, port: Int)
}

// androidMain — через NSD (Network Service Discovery)
actual class MdnsDiscovery actual constructor() {
    private val nsdManager: NsdManager by lazy {
        context.getSystemService(Context.NSD_SERVICE) as NsdManager
    }

    actual fun startDiscovery(serviceType: String, onFound: (String) -> Unit) {
        nsdManager.discoverServices(serviceType, NsdManager.PROTOCOL_DNS_SD,
            object : NsdManager.DiscoveryListener {
                override fun onServiceFound(service: NsdServiceInfo) {
                    nsdManager.resolveService(service, object : NsdManager.ResolveListener {
                        override fun onServiceResolved(info: NsdServiceInfo) {
                            onFound("${info.host.hostAddress}:${info.port}")
                        }
                        override fun onResolveFailed(service: NsdServiceInfo, errorCode: Int) {}
                    })
                }
                override fun onStartDiscoveryFailed(serviceType: String?, errorCode: Int) {}
                override fun onStopDiscoveryFailed(serviceType: String?, errorCode: Int) {}
                override fun onDiscoveryStarted(serviceType: String?) {}
                override fun onDiscoveryStopped(serviceType: String?) {}
                override fun onServiceLost(serviceInfo: NsdServiceInfo) {}
            })
    }
    // ...
}
```

### Вариант 3: USB-синхронизация

Для максимальной безопасности — синхронизация через USB-кабель, без участия сети:

```kotlin
// androidMain — через Android USB Host API
class UsbSync(private val usbManager: UsbManager) {

    fun findConnectedDevice(): UsbDevice? {
        return usbManager.deviceList.values.firstOrNull { device ->
            // Фильтр по VID/PID вашего приложения
            device.vendorId == 0x1234 && device.productId == 0x5678
        }
    }

    suspend fun syncOverUsb(device: UsbDevice, data: ByteArray) {
        val connection = usbManager.openDevice(device) ?: return
        val endpoint = device.getInterface(0).getEndpoint(0)
        connection.bulkTransfer(endpoint, data, data.size, TIMEOUT_MS)
    }
}
```

---

## 38.7. Сжатие данных (LZ4)

При синхронизации больших объёмов (история чата) важно сжимать данные перед передачей. **LZ4** — быстрый алгоритм сжатия с декомпрессией > 4 ГБ/с. Идеален для real-time приложений.

### KMP-библиотека для LZ4

[Kotlin-LZ4](https://github.com/lz4/lz4-java) — Java-реализация, работает на JVM/Android. Для iOS/Kotlin-Native можно использовать [lz4 через cinterop](https://github.com/lz4/lz4) или обернуть через expect/actual.

```kotlin
// commonMain
expect object Compression {
    fun compress(data: ByteArray): ByteArray
    fun decompress(data: ByteArray): ByteArray
}

// androidMain / jvmMain — через lz4-java
import net.jpountz.lz4.LZ4Factory

actual object Compression {
    private val factory = LZ4Factory.fastestInstance()
    private val compressor = factory.fastCompressor()
    private val decompressor = factory.fastDecompressor()

    actual fun compress(data: ByteArray): ByteArray {
        val maxCompressedLength = compressor.maxCompressedLength(data.size)
        val compressed = ByteArray(maxCompressedLength)
        val compressedLength = compressor.compress(data, 0, data.size, compressed, 0, maxCompressedLength)
        return compressed.copyOf(compressedLength)
    }

    actual fun decompress(data: ByteArray): ByteArray {
        // LZ4 не хранит оригинальный размер — нужно передавать его отдельно
        // Обычно в первые 4 байта записывают decompressedLength
        val decompressedLength = ByteBuffer.wrap(data.copyOfRange(0, 4)).int
        val restored = ByteArray(decompressedLength)
        decompressor.decompress(data, 4, restored, 0, decompressedLength)
        return restored
    }
}
```

> [!TIP]
> **Сжатие + шифрование — порядок важен!** Всегда: **сжать → зашифровать → отправить**. Не наоборот. Шифрование делает данные псевдослучайными — сжатие зашифрованных данных не даст никакого выигрыша (размер останется ~тем же). Сначала сжимаете открытый текст, потом шифруете сжатый результат.
>
> ```kotlin
> val compressed = Compression.compress(plaintext)  // 1. Сжатие
> val encrypted = AesGcm.encrypt(compressed, key)   // 2. Шифрование
> // отправка...
> ```

---

## 38.8. Push-уведомления в P2P-приложении

В P2P-мессенджере проблема: сообщение приходит по P2P, но если приложение закрыто — некому его принять. Решение — гибридный подход через Notification Service.

### Архитектура Notification Service (как в ReMatch Chat)

```
[Отправитель] ──P2P──→ [Получатель (если онлайн)]
       │
       └─если оффлайн─→ [Notification Service] ──→ [FCM/APNs] ──→ [Получатель]
```

1. Отправитель пытается отправить сообщение напрямую через P2P.
2. Если получатель оффлайн — отправитель шлёт **зашифрованное** уведомление на Notification Service.
3. Notification Service пересылает push через FCM (Android) / APNs (iOS).
4. Получатель получает push, открывает приложение, устанавливает P2P-соединение, скачивает сообщение.

> [!TIP]
> **Notification Service видит только зашифрованные данные.** Сам push содержит только идентификатор отправителя и зашифрованный payload. Реальное сообщение расшифровывается только на устройстве получателя. Notification Service не имеет доступа к ключам E2E.

```kotlin
// commonMain
class NotificationService(private val httpClient: HttpClient) {

    suspend fun sendPushNotification(
        recipientId: String,
        encryptedPayload: ByteArray,
        messageId: String
    ) {
        httpClient.post("https://notifications.example.com/send") {
            contentType(ContentType.Application.Json)
            setBody(buildJsonObject {
                put("recipient", recipientId)
                put("message_id", messageId)
                put("payload", encryptedPayload.toBase64())  // зашифровано E2E
            })
        }
    }
}

// Получатель (Android) — через FirebaseMessagingService
class RmcMessagingService : FirebaseMessagingService() {
    override fun onMessageReceived(message: RemoteMessage) {
        val encryptedPayload = message.data["payload"]?.fromBase64()
        val senderId = message.data["sender"] ?: return

        // Расшифровываем E2E своим приватным ключом
        val decrypted = e2eEncryption.decrypt(encryptedPayload, senderPublicKey, mySecretKey)

        // Показываем уведомление
        showNotification(senderId, decrypted.toString())
    }
}
```

---

## 38.9. Канальные сервера (федерация)

Вместо одного центрального Хаба можно позволить пользователям разворачивать свои сервера (как в Matrix, XMPP). Это называется **федерацией**.

```
[Пользователь A] ──→ [Хаб 1] ←──→ [Хаб 2] ←── [Пользователь B]
```

Пользователь A регистрируется на Хабе 1, пользователь B — на Хабе 2. Хабы общаются между собой по федеративному протоколу (XMPP, Matrix federation, или собственный).

### Преимущества федерации

- **Децентрализация** — нет единой точки отказа, нет единого владельца.
- **Суверенитет данных** — компания может развернуть свой Хаб и хранить данные в своей юрисдикции.
- **Совместимость** — пользователи разных Хабов могут общаться.

### Недостатки

- **Сложность** — нужно реализовать федеративный протокол, обработку сбоев, синхронизацию.
- **Согласованность** — если Хаб 1 недоступен, сообщения для его пользователей теряются (или ставятся в очередь).
- **Модерация** — каждый Хаб moderирует свой контент, что может привести к несогласованности.

### Простая федерация через HTTP

```kotlin
// Хаб 1 отправляет сообщение на Хаб 2 для пользователя B
class FederationClient(private val httpClient: HttpClient) {

    suspend fun sendMessageToRemoteHub(
        remoteHubUrl: String,
        recipientId: String,
        encryptedMessage: ByteArray
    ) {
        httpClient.post("$remoteHubUrl/api/federation/message") {
            header("X-Hub-Signature", signRequest(remoteHubUrl, encryptedMessage))
            contentType(ContentType.Application.Json)
            setBody(buildJsonObject {
                put("recipient", recipientId)
                put("payload", encryptedMessage.toBase64())
            })
        }
    }

    private fun signRequest(url: String, body: ByteArray): String {
        // Подпись запроса приватным ключом Хаба (HMAC-SHA256)
        // Получатель проверяет подпись публичным ключом отправляющего Хаба
        return Hmac.sha256(hubSecretKey, body).toHex()
    }
}
```

---

## 38.10. Чеклист production P2P-мессенджера

- [ ] **Сигналинг-сервер** (WebSocket через Ktor) для обмена ключами и ICE candidates.
- [ ] **STUN-сервер** (свой или публичный Google) для обнаружения публичных адресов.
- [ ] **TURN-сервер** (coturn на VPS) как fallback для симметричных NAT.
- [ ] **E2E-шифрование** (libsodium или Tink) для всех сообщений.
- [ ] **Safety numbers** для верификации публичных ключей собеседников.
- [ ] **Push-уведомления** через Notification Service (FCM + APNs).
- [ ] **Синхронизация между устройствами** через Посредника или mDNS/USB.
- [ ] **Сжатие данных** (LZ4) перед шифрованием и отправкой.
- [ ] **Ротация ключей** каждые N сообщений (Double Ratchet или упрощённый вариант).
- [ ] **Защита от replay-атак** (timestamp + nonce в каждом сообщении).
- [ ] **Обработка офлайна** (очередь неотправленных сообщений с retry).
- [ ] **Мониторинг качества соединения** (RTT, packet loss, выбор лучшего ICE candidate).

---

## 38.11. Дополнительные ресурсы

- [RFC 9000 — QUIC: A UDP-Based Multiplexed and Secure Transport](https://www.rfc-editor.org/rfc/rfc9000) — официальная спецификация QUIC.
- [RFC 8445 — ICE](https://www.rfc-editor.org/rfc/rfc8445) — алгоритм ICE.
- [RFC 8489 — STUN](https://www.rfc-editor.org/rfc/rfc8489) — протокол STUN.
- [RFC 8656 — TURN](https://www.rfc-editor.org/rfc/rfc8656) — протокол TURN.
- [Signal Protocol Documentation](https://signal.org/docs/) — спецификация Double Ratchet.
- [Cronet Documentation](https://developer.android.com/google/play/cronet) — Google Cronet для Android.
- [coturn на GitHub](https://github.com/coturn/coturn) — open-source TURN-сервер.
- [kotlin-multiplatform-libsodium](https://github.com/ionspin/kotlin-multiplatform-libsodium) — KMP-обёртка над libsodium.
- [Google Tink](https://github.com/tink-crypto/tink) — высокоуровневая крипто-библиотека.
- [Matrix Protocol](https://matrix.org/docs/) — референсная федеративная архитектура.

---

⬅️ [← Адаптивные лейауты](31-adaptive-layouts.md) | [К содержанию](README.md) | [Программирование с нуля →](00-programming-foundations.md) ➡️
